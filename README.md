# bountyplz – automated security reporting from markdown templates

### description

This is a project created by [Frans Rosén](https://twitter.com/fransrosen). The idea is to be able to submit a report without any interaction. It's taking advantage of all features the existing site has, such as attachments, inline images, assets, weaknesses and severity. On some weaknesses HackerOne asks additional questions, these are also supported.

bountyplz currently only supports submitting to HackerOne.

bountyplz will sign in to HackerOne and keep the session, create a draft and submit the report, all in one step. It also supports 2FA, if this is enabled on your HackerOne-account.

<img src="https://github.com/fransr/bountyplz/raw/documentation-files/preview/preview1.png" width="700" />

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
`-d` for draft-only

### usage Bugcrowd `bc`

Coming soon!

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
|`severity`|string|`none, low, medium, high, crical`|
|`url`|string|if weakness is Stored XSS, ClickJacking or CSRF, this one will be used in the Weakness-questions|
|`injection-type`|string|if weakness is SQL Injection, this one will be matched against a list of different injection types:<br /><code>Classic / In-Band, Out-of-Band, Blind / Inferential,</code><br /><code>UNION Operation, Boolean, Error based, Time delay</code>|

When the report is submitted, an additional `report`-attribute will be added to the markdown with the reference URL for the report. This is to make sure the same report is not submitted twice.

Asset, weakness and `injection-type` will try to match against the list of options. If multiple are found, a list will be shown to select the right one:

<img src="https://github.com/fransr/bountyplz/raw/documentation-files/preview/preview2.png" width="300" />

### impact

If any header with the word `impact` exist in the report, the report will be split in half and the content after Impact will be inserted in the Impact-field. If no Impact exists in the report, the Impact field will only contain a `#` rendering it empty.

```md
---
asset: example.com
---

# Report title

Report description

### impact

This will be in the impact field.
```

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

*Please note that Asset and Severity and Weakness-questions are not currently possible to save in the draft on HackerOne*.

### batch

This command will run all markdown files and report them. If a report already has a "report: "-reference in it, the report will not be sent.

```
find . -name "*.md" \( -exec bountyplz h1 <program> {} \; -o -quit \)
```

### todo

* Add Bugcrowd: `bountyplz bc mastercard reports/report.md`

