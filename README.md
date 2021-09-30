# Azure Host Plugin for HashiCorp Boundary

This repo contains a Host-type plugin for [HashiCorp
Boundary](https://www.boundaryproject.io/) allowing dynamically sourcing hosts
from Azure.

Host sets created with this plugin define filters which select and group like VM
resources within Azure; these host sets can in turn be added to targets within
Boundary as host sources.

At creation or update of a host catalog of this type, configuration of the
plugin is performed via the attribute/secret values passed to the create or
update calls actions. These values are input as JSON objects; the expected types
are indicated below with the valid fields.

Current limitations of this plugin:

- Only VMs can be added to host sets through this plugin, not any other type of
  compute resource
- A client secret must be generated against a registered application with any
  necessary permissions

## Credential Rotation

Although credentials are stored encrypted within Boundary, by default this
plugin will attempt to rotate credentials when they are supplied through the
`secrets` object on a create or update call to the host catalog resource. The
given credentials will be used to create a new credential, and then the given
credential will be revoked. In this way, after rotation, only Boundary knows the
client secret in use by this plugin.

*NOTE*: Azure's credentials are eventually consistent and there may be
significant delay (up to ten minutes or more) between when a credential is
created and is available for use, or when a credential is deleted and
unavailable for use. At the API level, this can be true even if the Azure Portal
indicates that the credential is (or isn't) there. This delay will manifest as
authentication failures when performing actions using this plugin. If secrets
have just been supplied and `disable_credential_rotation` is not specified,
please be patient and try again later.

## Attributes and Secrets

### Host Catalog

The following `attributes` are valid on an Azure host catalog resource:

- `disable_credential_rotation` (bool): If `true`, credential rotation will not
  be performed. See the Credential Rotation section above.
- `tenant_id` (string): The ID of the tenant. Within the Overview section of the
  Application Registration blade within the Azure Portal, this is listed as
  `Directory (tenant) ID`.
- `subscription_id` (string): The ID of the subscription, as found in the
  Subscriptions Blade within the Azure Portal.
- `client_id` (string): The ID of the client. Within the Overview section of the
  Application Registration blade within the Azure Portal, this is listed as
  `Application (client) ID`.

The following `secrets` are valid on an Azure host catalog resource:

- `secret_id` (string): Only necessary if rotation is enabled. The ID of the
  secret corresponding to the `secret_value`. Within the Certificates and
  Secrets section of the Application Registration blade within the Azure Portal,
  this is the full string corresponding to the `Secret ID` of a provisioned
  client secret.
- `secret_value` (string): The actual secret value. Within the Certificates and
  Secrets section of the Application Registration blade within the Azure Portal,
  this is the full string corresponding to the `Value` of a provisioned client
  secret.

### Host Set

The following attributes are valid on an Azure host Set resource:

- `filter` (string): The [Azure Resource Manager (ARM)
  filter](https://docs.microsoft.com/en-us/rest/api/resources/Resources/List)
  used to select the resources that should be a part of this host set