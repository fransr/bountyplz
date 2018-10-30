# bountyplz – automated security reporting from markdown templates

[![Rawsec's CyberSecurity Inventory](https://inventory.rawsec.ml/img/badges/Rawsec-inventoried-FF5050_flat.svg)](https://inventory.rawsec.ml/tools.html#bountyplz)
[![GitHub issues](https://img.shields.io/github/issues/fransr/bountyplz.svg)](https://github.com/fransr/bountyplz/issues)
[![GitHub forks](https://img.shields.io/github/forks/fransr/bountyplz.svg)](https://github.com/fransr/bountyplz/network)
[![GitHub stars](https://img.shields.io/github/stars/fransr/bountyplz.svg)](https://github.com/fransr/bountyplz/stargazers)
[![GitHub license](https://img.shields.io/github/license/fransr/bountyplz.svg)](https://github.com/fransr/bountyplz)

### description

This is a project created by [Frans Rosén](https://twitter.com/fransrosen). The idea is to be able to submit a report without any interaction. It's taking advantage of all features the existing site has, such as attachments, inline images, assets, weaknesses and severity.

bountyplz supports submitting to HackerOne and Bugcrowd.

bountyplz will sign in to HackerOne or Bugcrowd and keep the session, create a draft and submit the report, all in one step. It also supports 2FA, if this is enabled on your HackerOne- or Bugcrowd-account.

HackerOne:<br />
<img src="https://github.com/fransr/bountyplz/raw/documentation-files/preview/preview1.png" width="700" />

Bugcrowd:<br />
<img src="https://github.com/fransr/bountyplz/raw/documentation-files/preview/preview3.png" width="700" />

### install

```
brew install jq
brew install gnu-sed

ln -fs "$(pwd)/bountyplz" /usr/local/bin/bountyplz
```

### usage HackerOne `h1`

Place `.env` with `HACKERONE_USERNAME` and `HACKERONE_PASSWORD` next to the binary.

```
bountyplz h1 <program> <markdown-file>
```

`-p` for preview<br />
`-d` for draft-only<br />
`-f` for force

### usage Bugcrowd `bc`

Place `.env` with `BUGCROWD_USERNAME` and `BUGCROWD_PASSWORD` next to the binary.

```
bountyplz bc <program> <markdown-file>
```

`-p` for preview<br />
`-d` for draft-only (will upload files but not save any draft as this is currently not supported on Bugcrowd)<br />
`-f` for force

### howto

Write report in markdown, use frontmatter for attributes for the report. The title of the report will be taken from the content's first #-header.

```md
---
severity: high
weakness: xss reflected
asset: example.com
---

# Report title

Report description
```

The following attributes are currently supported:

| key   | type | desc |
|-------|------|---|
|`asset`|string|will be matched against the list of assets for the program|
|`weakness`|string|will be matched against the list of weaknesses for the program. |
|`attachments`|json-array|list of files that should be attached. `["test.jpg","test2.jpg"]`<br />if images and videos are used inline, these does not need to be in this list|
|`url`|string|bug URL (BugCrowd only, not required)|
|`severity`|string|`none, low, medium, high, crical` (HackerOne only)|

When the report is submitted, an additional `report`-attribute will be added to the markdown with the reference URL for the report. This is to make sure the same report is not submitted twice.

`asset` and `weakness` will try to match against the list of available options. If multiple results are found, a list will be shown to select the right one:

<img src="https://github.com/fransr/bountyplz/raw/documentation-files/preview/preview2.png" width="300" />

### impact

For HackerOne, if any header with the word `impact` exist in the report, the report will be split in half and the content after Impact will be inserted in the Impact-field. If no Impact exists in the report, the Impact field will only contain a `#` rendering it empty.

```md
---
asset: example.com
---

# Report title

Report description

### impact

This will be in the impact field.
```

For Bugcrowd, the whole report will be inside the Description-field.

### inline attachments

When referring to images or videos inside the report, use this format: `<img upload src="x.jpg" />`

Every image or video element containing `<img|video upload` will be extracted from the report and uploaded automatically. The location of the file referenced will always be relative to the markdown-file, and the preview before submitting will make sure all files exists.

### preview `-p`

You can preview the report before sending it using:

```
bountyplz h1 yahoo -p test/report1.md
```

This will not submit the report, but show you how the report was parsed.

### draft-only `-d`

You can submit the report as a draft only using:

```
bountyplz h1 yahoo -d test/report1.md
```

*Please note that Asset and Severity are not currently possible to save in the draft on HackerOne*.

### force `-f `

Whenever a file has been reported, the markdown-file is being modified to add a reference to the report-URL inside the frontmatter called `report: URL`. This is to prevent the report from being submitted again. By using `-f` you can force the report to be submitted, even if it has a `report:`-entry in the frontmatter. Use with caution to prevent duplicate reports.

### batch

This command will run all markdown files and report them. If a report already has a "report: "-reference in it, the report will not be sent.

```
find . -name "*.md" \( -exec bountyplz h1 <program> {} \; -o -quit \)
```
