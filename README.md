# Xbox-WebAPI

[![PyPi - latest](https://pypip.in/version/xbox-webapi/badge.svg)](https://pypi.python.org/pypi/xbox-webapi/)
[![Documentation status](https://readthedocs.org/projects/xbox-webapi-python/badge/?version=latest)](http://xbox-webapi-python.readthedocs.io/en/latest/?badge=latest)
[![Build status](https://img.shields.io/github/workflow/status/OpenXbox/xbox-webapi-python/build?label=build)](https://github.com/OpenXbox/xbox-webapi-python/actions?query=workflow%3Abuild)
[![Discord chat channel](https://img.shields.io/badge/discord-OpenXbox-blue.svg)](https://openxbox.org/discord)

Xbox-WebAPI is a python library to authenticate with Xbox Live via your Microsoft Account and provides Xbox related Web-API.

Authentication is supported via OAuth2.

- Register a new application in [Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
  - Name your app
  - Select "Personal Microsoft accounts only" under supported account types
  - Add <http://localhost/auth/callback> as a Redirect URI of type "Web"
- Copy your Application (client) ID for later use
- On the App Page, navigate to "Certificates & secrets"
  - Generate a new client secret and save for later use

## Dependencies

- Python >= 3.6
- Libraries: aiohttp, appdirs, ms_cv, pydantic, urwid, yarl, ecdsa

## How to use

Install

```text
pip install xbox-webapi
```

Authentication

```text
# Note: you must use non child account (> 18 years old)
# 
# Token save location: If tokenfile is not provided via cmdline, fallback
# of <appdirs.user_data_dir>/tokens.json is used as save-location
#
# Specifically:
# Windows: C:\\Users\\<username>\\AppData\\Local\\OpenXbox\\xbox
# Mac OSX: /Users/<username>/Library/Application Support/xbox/tokens.json
# Linux: /home/<username>/.local/share/xbox
#
# For more information, see: https://pypi.org/project/appdirs and module: xbox.webapi.scripts.constants

xbox-authenticate --client-id <client-id> --client-secret <client-secret>
```

Example: Search Xbox Live via cmdline tool

```text
  # Search Xbox One Catalog
  xbox-searchlive "Some game title"
```

API usage

```py
import sys
import asyncio
from aiohttp import ClientSession
from xbox.webapi.api.client import XboxLiveClient
from xbox.webapi.authentication.manager import AuthenticationManager
from xbox.webapi.authentication.models import OAuth2TokenResponse
from xbox.webapi.common.exceptions import AuthenticationException
from xbox import *
client_id = 'YOUR CLIENT ID HERE'
client_secret = 'YOUR CLIENT SECRET HERE'
"""
For doing authentication, see xbox/webapi/scripts/authenticate.py
"""
async def async_main():
    tokens_file = "./tokens.json" # replace with path in auth scrip or just paste file with tokens here
    async with ClientSession() as session:
        auth_mgr = AuthenticationManager(
              client_id, client_secret, "", client_session=session
        )

        try:
            with open(tokens_file, mode="r") as f:
                  tokens = f.read()
            auth_mgr.oauth = OAuth2TokenResponse.parse_raw(tokens)
        except FileNotFoundError:
            print(f'File {tokens_file} isn`t found or it doesn`t contain tokens!')
            exit(-1)
        
        try:
              await auth_mgr.refresh_tokens()
        except ClientResponseError:
              print("Could not refresh tokens")
              sys.exit(-1)

        with open(tokens_file, mode="w") as f:
              f.write(auth_mgr.oauth.json())
        print(f'Refreshed tokens in {tokens_file}!')

        xbl_client = XboxLiveClient(auth_mgr)

        # Some example API calls
        # Get friendslist
        friendslist = await xbl_client.people.get_friends_own()
        print('Your friends:')
        print(friendslist)
        print()

        # Get presence status (by list of XUID)
        presence = await xbl_client.presence.get_presence_batch(["2533274794093122", "2533274807551369"])
        print('Statuses of some random players by XUID:')
        print(presence)
        print()

        # Get messages
        messages = await xbl_client.message.get_inbox()
        print('Your messages:')
        print(messages)
        print()

        # Get profile by GT
        profile = await xbl_client.profile.get_profile_by_gamertag("SomeGamertag")
        print('Profile under SomeGamertag gamer tag:')
        print(profile)
        print()

asyncio.run(async_main())
```

## Contribute

- Report bugs/suggest features
- Add/update docs
- Add additional xbox live endpoints

## Credits

This package uses parts of [Cookiecutter](https://github.com/audreyr/cookiecutter)
and the [audreyr/cookiecutter-pypackage](https://github.com/audreyr/cookiecutter-pypackage) project template.
The authentication code is based on [joealcorn/xbox](https://github.com/joealcorn/xbox)

Informations on endpoints gathered from:

- [XboxLive REST Reference](https://docs.microsoft.com/en-us/windows/uwp/xbox-live/xbox-live-rest/atoc-xboxlivews-reference)
- [XboxLiveTraceAnalyzer APIMap](https://github.com/Microsoft/xbox-live-trace-analyzer/blob/master/Source/XboxLiveTraceAnalyzer.APIMap.csv)
- [Xbox Live Service API](https://github.com/Microsoft/xbox-live-api)

## Disclaimer

Xbox, Xbox One, Smartglass and Xbox Live are trademarks of Microsoft Corporation. Team OpenXbox is in no way endorsed by or affiliated with Microsoft Corporation, or any associated subsidiaries, logos or trademarks.
