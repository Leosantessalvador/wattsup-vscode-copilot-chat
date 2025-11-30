# Wattsup with Github Copilot

*This is a fork of [vscode-copilot-chat](https://github.com/microsoft/vscode-copilot-chat), the official VS Code extension for Github Copilot Chat.*

Wattsup with Github Copilot allows you to monitor AI requests usage with the integrated Wattsup dashboard, giving quick access to key metrics such as output token usage and CO2 emission equivalences over different time periods.

![Wattsup preview screenshot](https://github.com/ucodia/wattsup-vscode-copilot-chat/blob/wattsup/wattsup-screenshot.png?raw=true)

## Installation

### From VS Code

Since this extension cannot be released in the VS Code marketplace, you will need to download the [latest release](https://github.com/Ucodia/wattsup-vscode-copilot-chat/releases) and install it manually by using the "Install from VSIX" command in VS Code.

### From command line

```
wget https://github.com/Ucodia/wattsup-vscode-copilot-chat/releases/download/v99.6.1/wattsup-copilot-chat-99.6.1.vsix
code --install-extension wattsup-copilot-chat-99.6.1.vsix
```

## Development

### Code organization

Since this is a fork of the official VS Code extension, the changes to common files has been limited and most of the logic lives in [src/wattsup](src/wattsup) in order to continue syncing this fork with the source.

### Core logic

The core logic is bootstraped by the [WattsupDashboard](src/wattsup/wattsupUsageDatabase.ts) web view. The extension periodically checks the [RequestLogger](src/platform/requestLogger/node/requestLogger.ts) requests list to collect AI request information. This data is then mapped to the closest model currently available in [EcoLogits](https://ecologits.ai/) calculator to estimate the energy consumption and environmental impacts of AI requests.

### Data storage and analytics

Data storage is defined in the [WattsupUsageDatabase](src/wattsup/wattsupUsageDatabase.ts) class. Current implementation relies on a singular `usage.csv` file stored in the extension storage folder. The file is locked before writes and only appended to guarantee consistency across VS Code instances. [arquero](https://github.com/uwdata/arquero) memory tables are used to compute analytics. This may not scale over time and require using a more robust data source engine such as [SQLite](https://github.com/sqlite/sqlite-wasm).

### Packaging and quirks

The official `vscode-copilot-chat` extension from which this repository is forked, relies on another non open source `vscode-copilot` extension. This makes it impossible to have both the Wattsup version of the extension and the official extension together. As such this fork was versioned differently starting with version `99.x.x` to avoid conflict with the official release.

Microsoft team confirmed that they are actively working on open sourcing the remaining bits from closed source component (see [GitHub issue](https://github.com/microsoft/vscode/issues/258742)).

If you want to build this from source, make sure to switch the [package.json](package.json#L10) `buildType` is set to `prod` before running `npm run package -- --allow-package-secrets sendgrid`.

### Updating models data

Wattsup relies on Ecologits to estimate energy usage and CO2 emissions. As such there are data files that may need to be synchronized from time to time.

Data can be updated such as:

```
curl https://raw.githubusercontent.com/genai-impact/ecologits/refs/heads/main/ecologits/data/models.json > src/wattsup/data/models.json
curl https://raw.githubusercontent.com/genai-impact/ecologits/refs/heads/main/ecologits/data/electricity_mixes.csv > src/wattsup/data/electricity_mixes.csv
npx csvtojson --checkType=true src/wattsup/data/electricity_mixes.csv > src/wattsup/data/electricity_mixes.json
rm src/wattsup/data/electricity_mixes.csv
```

### Synchronizing upstream releases

In order to properly sync commits from an upstream release, we first need to fetch the tags, create a local branch for that tag and then merge the commit in the target `wattsup` branch:

```
# 1. Fetch new upstream tag
git fetch upstream --tags

# 2. Create/update local branch that matches the upstream tag
git switch -C upstream-v0.34.0 v0.34.0

# 3. Merge upstream release into your branch
git switch wattsup
git merge --no-ff upstream-v0.34.0
```

## License

This project is released under the **MIT License**.
See the `LICENSE` file at the root of the repository for the full text.

### Third-Party Components

This project includes source code originally developed by **Ecologits**, distributed under the **Mozilla Public License 2.0 (MPL-2.0)**.

All MPL-2.0–licensed files include the appropriate license header, and the full text of the MPL-2.0 license is available in `LICENSE-MPL-2.0`.