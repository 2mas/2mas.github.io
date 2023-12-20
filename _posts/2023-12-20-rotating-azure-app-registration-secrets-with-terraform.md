---
title: "Rotating Azure App registration secrets with terraform"
excerpt: "I'll describe how to rotate App registration secrets in terraform"
date: 2023-12-20T08:00:30+02:00
categories:
  - blog
tags:
  - Terraform
  - Azure
---

Hey everyone, I thought I'd share my take on rotating secrets belonging to an Azure App Registration with terraform since it's not that easy to get it right. If you are reading this I will assume that you are familiar with App registrations already and won't explain more on the topic itself.

Different authentication methods is also another discussion where password-less authentication with managed identities would be a better choice generally, but this is not always doable.

### The problem

So let's have a look at the task of handling secrets, there are some important requirements to think about when doing this:

- A secret must not expire while being used, otherwise the application suffers downtime.
- A newly created secret must get used in an application before the old one gets deleted, otherwise a running application will use a deleted secret and suffers downtime.

When looking at the requirements above and given that we are trying to do this with terraform, it's clear that we need to have at least two valid secrets present during a rotation, because there will always be a delay to have a currently running app pick up a newly created secret when the old one gets deleted.

### Ways to solve the problem

There are some ways to do this and I'll describe some of them here, and in the end I will suggest my approach.

- **Monitor the secrets and keep track of expiration manually**

  You can use some automation to get a list of secrets that will expire. Then you have to find out what applications are actually using them, create a new secret, configure the applications to use the new secret and maybe deploy the applications with the new secret and optionally remove the old ones. This is time consuming and will probably be missed sooner or later.
- **Terraform - re-creation of a single secret based on a `time_rotating` resource.**

  An approach I've seen in a few places is to tie the secret to a [time_rotating](https://registry.terraform.io/providers/hashicorp/time/latest/docs/resources/rotating) resource like this:

  ```terraform
  resource "time_rotating" "rotation" {
    rotation_months = 3
  }

  resource "azuread_application_password" "secret" {
    application_id = azuread_application.app.id

    rotate_when_changed = {
      rotation = time_rotating.rotation.id
    }
  }
  ```
  It will re-create the resource when the `time_rotating` resource has been recreated. This approach would work if you didn't have a running application using the secret that will be destroyed. So unfortunately it breaks a running application during the time it would take to get the new secret out to the running application, so it's not an acceptable solution for me.
- **My take - Terraform with overlapping secrets**

  This builds on the previous example but adds a couple of important pieces using two secrets that always have overlapping rotation, the secret with the latest rotation-time will be used (in this case as a Key Vault Secret):

  ```terraform
  resource "time_rotating" "first" {
    rotation_months = 12
  }

  # Expires at time_rotating.first + 6 months
  resource "time_rotating" "second" {
    # This is the base timestamp to use
    rfc3339         = time_rotating.first.rotation_rfc3339
    rotation_months = 6

    lifecycle {
      ignore_changes = [rfc3339]
    }
  }

  # Secret is valid 24 months but will be rotated after 6 months
  resource "azuread_application_password" "rotating1" {
    display_name      = "terraform-generated-rotating-1"
    application_id    = azuread_application.app.id
    end_date_relative = "17520h" # Valid for 2 years

    rotate_when_changed = {
      rotation = time_rotating.first.id
    }
  }

  # Secret is valid 24 months but will be rotated after 6 months
  resource "azuread_application_password" "rotating2" {
    display_name      = "terraform-generated-rotating-2"
    application_id    = azuread_application.app.id
    end_date_relative = "17520h" # Valid for 2 years

    rotate_when_changed = {
      rotation = time_rotating.second.id
    }
  }

  # Use the secret with the latest expiration, here it's put into an Azure Key Vault
  resource "azurerm_key_vault_secret" "sp_secret" {
    name         = "clientsecret"

    # Make decision on which secret to use by checking the unix timestamp of each rotation timestamp
    value        = time_rotating.first.unix > time_rotating.second.unix ? azuread_application_password.rotating1.value : azuread_application_password.rotating2.value
    key_vault_id = azurerm_key_vault.app.id
  }
  ```

  **Explanation**:

  The trick here is to pick correct time-intervals to get a large enough window (I'll call it sync-window) to get the new secret deployed/synced into any running applications, and to base the second `time_rotating` resource on the first ones expiration timestamp and add 6 months to it. It means that it will correct any drifting every time the second rotation happens and you will always have a sync-window for about 6 months to update any application.

  Why it this important? Firstly let's imagine that we didn't do this step with basing the second resource on the first one. What happens then is that because terraform will be executed by some interaction (vs running in a control-plane where there is an infinite loop always running, for example in k8s) and this can happen irregularly, there will be drifting on the timestamps. For example if we have a `time_rotating` resource that is set to rotate in 6 months, and we run `terraform apply` after 7 months, the next rotation will have 7 months + 6 months as the new rotation date and not 6 + 6. This drift adds insecurity to the process because we have to keep an eye on these rotation timestamps because we don't know for sure when they will happen. Eventually the two resources would probably get rotated in the terraform run, and that means both secrets gets destroyed at the same time and the app suffers downtime.

  The algorithm for the above solution is like this:

  - Initial run:`time_rotating.first` gets rotation date 12 months from now,`time_rotating.second` gets rotation date 12 + 6 months from now since its based on`time_rotating.first`. The secret that gets stored in key vault will be the one connected to`time_rotating.second` because it has the latest expiration.
  - Run after 12 months:`azuread_application_password.first` will be rotated since it's connected to`time_rotating.first`. The new expiration date will be 12 months from this date. Remember that there can still be drifting here, the run may happen later than 12 months, but it's OK since we have another 6 months before both secrets would get rotated at the same time. Secret in`azuread_application_password.first` will be set to the key vault.
  - Run after 18 months:`azuread_application_password.second` will be rotated since it's connected to`time_rotating.second`, this one had an expiration of 12 + 6 months from the initial run. The new expiration date will be 6 months added on the expiration date of`time_rotating.first`, regardless of when the previous run was made. Secret in`azuread_application_password.second` will be set to the key vault.
  - Run after 24 months:`azuread_application_password.first` will be rotated and will get a new rotation date 12 months from now. Secret in`azuread_application_password.first` will be set to the key vault.
  - And so on (next rotation is for`azuread_application_password.second` after 6 more months)...

  ... and it continues like this, every 6th month the secret set in key vault will change and a new rotation will happen. Only on the initial run will the secret be used for 12 months. You could lower the rotation-intervals if you wish but try to keep the ratio, it depends on how large window you need. The important piece here is that every time the second secret is rotated it will sync up with the first one to always create a large enough sync-window to get the secret out to the application.

Let me know if you found this post useful, happy coding
