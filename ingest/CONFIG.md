# Provider configuration file

### Global default settings

`"ref"` is the git refernece (branch name, commit hash, tag) by which MDS is referenced.

```json
{
    "DEFAULT": {
        "ref": "master"
    },
    ...
}
```

### Provider-specific settings

Each provider block is keyed by the provider's UUID, which can be found here: https://raw.githubusercontent.com/CityOfLosAngeles/mobility-data-specification/master/providers.csv

Create as many provider sections as needed.

###### For auth token flow

To specify the authorization token header to send, fill in the `"auth_type"` and `"token"` fields (e.g. `Authorization: <auth_type> <token>`)

Optional fields:
* `"mds_api_url"` if the standard mds_api_url found here https://raw.githubusercontent.com/CityOfLosAngeles/mobility-data-specification/master/providers.csv needs to be overridden.
* `"mds_api_suffix"` for custom paths specified by MDS providers
* `"headers"` for additional headers needed for the MDS API request

Example config:

```json
{
    "DEFAULT": {
        "ref": "master"
    },
    "6d9ec5e6-8110-4ad7-a5f2-7a2ce45bc1de": {
        "auth_type": "Bearer",
        "token": "1234567890",
        "mds_api_suffix": "agency_xyz",
        "headers": {
            "optional-custom-header": "1.2.3"
        }
    }
}
```

###### For OAuth flow

This Provider example uses the more complex OAuth client_credentials style.

Usually this involves registering an application with the Provider's platform, which generates the client_id and client_secret below. Talk to the Provider for more details.

Optional fields:
* `"headers"` for additional headers needed for the MDS API request

```json
{
    "DEFAULT": {
        "ref": "master"
    },
    "72e0b5f4-de11-4b9f-b52f-31fde095ff2f": {
        "token_url": "https://developers.providerB.co/api/token",
        "client_id": "IjbPIvLWUMej7RMbUykH",
        "client_secret": "x0N9e805EGfzfmqDTlnrNhRxub09bllRpLKKE64E",
        "scope": "mds.status_changes,mds.trips"
    }
}
```
