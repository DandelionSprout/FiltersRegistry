# AG Filters Registry

This repository contains the known filters subscriptions available to AdGuard users. We re-host these filters on `filters.adtidy.org`. Also, these filters can be slightly modified in order to achieve better compatibility with AdGuard.
## What filters can be added to this repository

We may add third-party filters to AdGuard Filters Registry. When making a decision about adding a third-party filter, we follow these rules:

1. The filter should be oriented towards browser content blockers.
2. The filter should be legal. If it has rules for paywall circumvention, we won't add such a filter.
3. The filter should have a place for receiving user complaints and holding discussions, such as a repository on github.com, or a website open to public.
4. The filter should be relatively popular, meaning:
    * if there is a repository on GitHub, the number of stars should be at least 50
    * if there is no repository on GitHub, the number of analyzed issues and discussions is estimated at 10 per month on the filter's website
    * the filter should be actively supported for at least 6 months
5. The filter should be regularly updated with at least 10 updates per month.
6. The filter should be compatible with AdGuard products. You can familiarize yourself with AdGuard syntax here: https://adguard.com/kb/general/ad-filtering/create-own-filters/.
7. If the filter works only in some operating systems and satisfies all other criteria, it will be added but only for the supported platforms.
8. Previously added filters that haven't received any support for a year will be removed. We reserve the right to remove the filter earlier, depending on circumstances.
9. If the filter contains too many problematic rules, it will not be added. A rule is considered problematic if it causes false positives or otherwise displays unintended behavior. Decisions about filters with problematic rules are arbitrary and there may be exceptions (see items 9 and 10, for example).
    * If the filter intentionally blocks or restricts access to any services for no reason other than being a reflection of the filter author's opinion, the filter will not get added, or will get removed if already added.
10. If the filter is popular in a specific region and there are no alternatives to it, then it can be added as is.
11. If the filter gets added, it receives a so-called [trustLevel](#trustLevel) (Low, High, Full), based on the number of problematic rules it contains and some other factors. Filters without "Full" trust level may have part of their rules disabled.
    * The trust level of a filter can be re-reviewed and raised if the author improves the filter over time.
12. If there are two or more similar filters that satisfy all other criteria, they all may be added if they don't duplicate each other and don't conflict with each other. If there is a large amount of conflicting or duplicate rules, the filter with more matches on such rules gets the priority.
## Filters metadata

- `template.txt`

    [Template file](#templates) is used by the filters compiler to prepare the final filter version.

- `exclude.txt`

    A list of regular expressions. Rules that match these exclusions will not be included in the resulting filter.

- `metadata.json`

    Filter metadata. Includes name, description, etc.

    * `filterId` — unique filter identifier (integer)
    * `name` — filter name; can be localized
    * `description` — filter description
    * `timeAdded` — time when this filter was added to the registry; milliseconds since January 1, 1970; you can exec `new Date().getTime()` in the browser console to get the current time
    * `homepage` — filter website/homepage
    * `expires` — filter's default expiration period; used as filter update interval if "Default" is chosen for according setting in AdGuard product
    * `displayNumber` — this number is used when AdGuard sorts available filters (GUI)
    * `groupId` — [group](#groups) identifier
    * `subscriptionUrl` — default filter subscription URL
    * `tags` — a list of [tags](#tags)
    * <a id="trustLevel"></a> `trustLevel` — level of trust which describe [allowed and permitted rules types](https://github.com/AdguardTeam/FiltersCompiler/tree/master/src/main/utils/trust-levels); possible values:
        * `low` — only low-risk rule types are allowed; defaults to **low** if trust level is not configured at all
        * `high` — trusted third-party filter lists; some particular rules from there are still permitted
        * `full` — all types of filter rules are allowed; only AdGuard filter lists have full trust at the moment
    * `platformsIncluded` — [the list of platforms](https://adguard.com/kb/general/ad-filtering/create-own-filters/#platform-and-not_platform-hints) to compile the filter for, e.g. `["mac", "windows", "android"]`. If you need to compile the filter for all platforms remove this property
    * `platformsExcluded` — [the list of platforms](https://adguard.com/kb/general/ad-filtering/create-own-filters/#platform-and-not_platform-hints) to skip while filter compiling, e.g. `["ios", "ext_safari"]`. If you need to compile the filter for all platforms remove this property

    > Note please that both `platformsIncluded` and `platformsExcluded` should not be set in filter's metadata simultaneously.

    <details>
      <summary>Metadata example</summary>

    ```json
    {
      "filterId": 2,
      "name": "AdGuard Base filter",
      "description": "EasyList + AdGuard English filter. This filter is necessary for quality ad blocking.",
      "timeAdded": 1404115015843,
      "homepage": "https://adguard.com/kb/general/ad-filtering/adguard-filters/",
      "expires": "4 days",
      "displayNumber": 1,
      "groupId": 1,
      "subscriptionUrl": "https://filters.adtidy.org/extension/chromium/filters/2.txt",
      "tags": [
        "purpose:ads",
        "reference:101",
        "recommended",
        "reference:2"
      ],
      "trustLevel": "full",
      "platformsIncluded": [
        "windows",
        "mac",
        "android",
        "ext_ublock"
      ]
    }
    ```
    </details>

- `revision.json`

  Filter version metadata, automatically filled and overwritten on each build.

- `filter.txt`

  Resulting compiled filter.

- `diff.txt`

  Build log that contains excluded and converted rules with an explanation.

### <a id="tags"></a> Tags

Every filter can be marked by a number of tags. Every tag metadata listed in `/tags/metadata.json`.

<details>
  <summary>Example</summary>

```json
{
    "tagId": 1,
    "keyword": "purpose:ads"
  },
```
</details>

Possible tags:
* `lang:*` — for language-specific filters; one or multiple lang-tags can be used. For instance, AdGuard Russian filter is marked with the `lang:ru` tag.

* `purpose:*` — determines filters purposes; multiple purpose-tags can be used for one filter list. For instance, `AdGuard DNS` is marked with both `purpose:ads` and `purpose:privacy`.

* `recommended` — for low-risk filter lists which are recommended to use in their category. The category is determined by the pair of the `lang:*` and `purpose:*` tags.

* `obsolete` — for abandoned filter lists; filter's metadata with this tag will be excluded from `filters.json` and `filters_i18n.json`.
### <a id="groups"></a> Groups

`/groups/metadata.json` — filters groups metadata. Each filter should belong to one of the groups.

## Filters optimization

For each filter, AdGuard compiles two versions: full and optimized. Optimized version is much more lightweight and does not contain rules which are not used at all or used rarely. Rules usage frequency comes from the collected [filter rules statistics](https://adguard.com/kb/general/ad-filtering/tracking-filter-statistics/) (thanks to the volunteers who enabled it in their AdGuard).

* `script/optimization_config.json` - defines the target for the optimization process. AdGuard will try to compress the lists by removing the most rarely used rules until the compression goal (defined in percents) is met.

## Filters compiler customization

`yarn custom_platforms` customizes the way filters are compiled for certain platforms. We should use it if we need to
temporary change rules for a platform. In all other cases, we should prefer the default configuration.
Below is a example of the configuration for the platform `AdGuard for Chrome` with comments:

```javascript
    "EXTENSION_CHROMIUM": {
        // Defines the platform for which the settings are specified.
        "platform": "ext_chromium",
        // Defines the path that can be used to access the settings or resources associated with this platform.
        "path": "extension/chromium",
        // Overrides the expires value set in the filter metadata (for this platform).
        "expires": "12 hours",
        "configuration": {
            // Sets an array of regular expressions that will be used to remove certain rules.
            "removeRulePatterns": [
                "^((?!#%#).)*\\$\\$|\\$\\@\\$",
                "\\$(.*,)?replace=",
                "important,replace=",
                "\\$(.*,)?app",
                "\\$network",
                "\\$protobuf",
                "important,protobuf",
                "\\$extension",
                ",extension"
            ],
            // Sets an array of objects that will be used to replace certain values.
            "replacements": [
                {
                    "from": ":has\\(",
                    "to": ":-abp-has("
                }
            ],
            // Specifies whether to ignore hints for rules. A value of "false" means that hints will not be ignored.
            "ignoreRuleHints": false
        },
        "defines": {
            "adguard": true,
            "adguard_ext_chromium": true
        }
    },
```

## Filters localization

If you want to help with filters translations, you can join us on Crowdin: https://crowdin.com/project/adguard-applications/en#/miscellaneous/filters-registry

Please learn more about translating our products: https://adguard.com/kb/miscellaneous/contribute/translate/program/

## <a id="templates"></a> Filters templates
`@include` directive allows to include the content of specified file into the filter.

More information about the `@include` directive and its options can be found here: https://github.com/AdguardTeam/FiltersCompiler/#include-directive.

## How to build

```
yarn install
```

Build filters and patches:
```
yarn build && yarn build:patches
```

Build with white/black lists:
```
yarn build -i=1,2,3 -s=4,5,6 && yarn build:patches
```

Validate `filters.json` and `filters_i18n.json` for platforms:
```
yarn validate:platforms ./platforms
```

> For AdGuard filters **all locales** are required, it means 100% translated.

Validate locales:
```
yarn validate:locales
```

> For third-party filters only `REQUIRED_LOCALES` should be 100% done.

## Compress Repository

Once a year, we will compress the repository to reduce its size. We will delete all remote branches and overwrite the master branch with a squashed history. The compression script will retain the first 10,000 commits in their original order in the history. All other commits (except the first one) will be squashed into a single commit.
