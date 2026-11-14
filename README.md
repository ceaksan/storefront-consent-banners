# Consent banners on ecommerce storefronts: dataset v1

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24090344.svg)](https://doi.org/10.5281/zenodo.24090344)

How ecommerce storefronts present cookie consent on their homepage, on desktop and on
mobile, as a first-time visitor sees it. One row per site and viewport. v1 covers
storefronts on three Turkish SaaS ecommerce platforms; later releases add other
platforms and markets, told apart by `market` and `site_language`.

This datasheet ships with the dataset. It describes only what the file contains and how
each column was measured. No row names a website; matching a row to a site takes
outside knowledge of that site.

## At a glance

- 1587 storefronts, 3174 rows (desktop and mobile).
- Three platforms and the consent banner vendors are coded (`platform_A`,
  `vendor_61`, ...). Codes are random and carry no order or size.
- Banner screenshots taken 3 to 9 November 2026; site profiles for the sector from
  homepage scans of late August 2026.
- Licence: CC BY 4.0.

## Sample

Every site in v1 comes from the public customer (reference) lists that the three
platforms publish. These lists favour larger, better maintained stores, so the dataset
describes the stores the platforms show off, not every store on those platforms.

- **Platform** is the one whose list the site came from. A site that has since moved to
  another platform keeps that code and has `platform_moved` set to true.
- **Origin** is `reference_list` on every v1 row. Later releases will add sites met
  while browsing (`browsing`), further platforms and further vendors.
- **Market** is the market of the platform list a site came from: `TR` on every v1 row.
  A few of these stores sell abroad; their declared language shows it.
- **Site language** is what the homepage declares in `<html lang>`, not a detected
  language: `tr`, `en` or `other` (rare languages grouped so they do not single a site
  out), empty when the page declares none.
- **Sector** is described in its own section below.

## Sector

Our own 21-category taxonomy, not the platforms' labels; each category's definition is
in `columns.json` (`sector_definitions`).

- **Input:** a short text profile of each homepage: title, meta description and
  keywords, site name, first heading, and the header and menu links, which on a shop
  homepage are mostly the category menu.
- **Classification:** two open-weight models run locally, Qwen 3.5 35B-A3B and Gemma 4
  31B (both Q8_0, through Ollama, temperature 0). Each received the category list with
  definitions and the profile, and answered with a primary category, an optional second
  one and a confidence score.
- **Decision:** where both models gave the same primary category, that category was
  kept (about 91 % of sites). For the rest, a person read the profile and decided. Two
  sites have no sector because no profile could be built from their homepage.
- **Check:** a platform files 1131 of the sites under one of its own sectors; 1130 of
  them have a final category, and on 1030 of those (91 %) the final category or the
  second one matched that label.

## Collection

Each homepage was opened once per viewport in a fresh browser with no stored cookies,
and the first screen was captured before any interaction.

- Desktop 1041 x 750, mobile 390 x 844 with a phone user agent and touch.
- Banners that did not show to the first automated pass (late loading, hidden from
  automated browsers) were captured again: by a second automated pass, by hand in a
  regular browser, or through a desktop automation agent driving a regular browser.
- Homepage only. No login, no form submitted, no consent choice made for this dataset.
- Automated passes also stored the page's own record of each banner (computed colours,
  font size and weight, box positions) with the screenshot; captures made by hand have
  none.

## Labelling

One person labelled every screenshot in a purpose-built tool: the banner, its backdrop,
title, text and each control (accept, reject, settings, close), plus the banner type.
Automatic detection proposed boxes; the person accepted, corrected or redrew each one.
There was no second labeller, so agreement between labellers was not measured.

**Banner type**

- `none`: no consent banner on the first screen.
- `info`: a cookie notice that offers no choice (no reject, no settings).
- `cmp`: a banner that offers a choice: reject, settings, or both.
- `both`: a choice banner and a separate notice on the same page.

A control counts as present on a row when the centre of its box lies inside the banner
on that screenshot. A control pushed off the screen at one viewport is absent on that
row, because a visitor at that viewport does not see it.

**Vendor** was decided from evidence on the page (the banner's own container first,
then markup, network requests and cookies) and checked by hand. `vendor_kind` says
whether the banner is the platform's own (`built_in`), a consent management platform
(`third_party_cmp`), something else, or could not be told.

## Measurements

**Colour.** A region's colour is its dominant colour: pixels grouped at 4 bits per
channel, the fullest group averaged. This keeps label and border pixels from greying a
fill. Fills come from the screenshot; label colour, font size and weight come from the
page's own record of the same element, used only where its background agrees with the
screenshot. Where the capture visit left no such record, font size and weight come from
a second visit that found each button by its label; a check on 20 banners measured both
ways gave the same size on 34 of 36 buttons. A button's _face_ is its fill when filled,
else its border or label colour. Colour difference is CIEDE2000; contrast is the WCAG
ratio. Theme, temperature and hue family are coarse names for the same colours.

**Prominence.**

- `banner_page_de`: the banner against the page as seen in the screenshot, backdrop
  included.
- `banner_ring_de`: the banner against a 40 px ring of page right around it.
- `saliency_lift`: how much the banner stands out from its surroundings on a spectral
  residual saliency map.
- `nudge`, a rule-based visual tendency, not a measured effect on visitors: a filled
  button outweighs an outline one; otherwise the button 10 or more further from the
  banner colour (CIEDE2000) is favoured, and failing that the one at least 1.5 times
  larger by box area, unless the other is 10 or more further from the banner colour;
  otherwise `equal`. Where the reject is a link inside the text, the link's box is
  compared the same way, so a filled accept button outweighs it.

**Accept against reject.** Only where the reject is a button. A reject offered as a link
inside the text is set in the text's type and is not compared with a button. Two faces
within a CIEDE2000 difference of 10 count as the same colour.

**Wording.** Button labels were read from the screenshot (on-device text recognition)
or from the page, site names masked, and sorted into a few categories by keyword. Only
the category is in the file, and only on rows where that control is present.

## Coarsening

To make it harder to match a row to a site by its exact measurements:

- `banner_area_share` rounded to 0.05, `saliency_lift` to 0.25;
- colour differences as bands 0-5, 5-10, 10-20, 20-40, 40+;
- contrast ratios as bands <3, 3-4.5, 4.5-7, 7+;
- vendors on fewer than 5 sites grouped as `vendor_rare`.

This lowers the risk; it does not rule matching out. On the desktop rows, 406 of the
1587 sites are still alone on the combination of platform, sector, vendor, position,
hue family, area, nudge, wording and site language. Every column describes something
any visitor can see on the site, so a matched row tells someone who knows the site
little that a visit would not; no row carries what a site tracks.

## Not in the dataset

Domains and URLs, screenshots, banner text and button labels, cookie names, colour
values, tag manager data, and anything about what a site tracks before or after a
choice.

## Columns

3174 rows, 44 columns. Empty means not applicable or not measured on that row.

| Column | Meaning | Filled | Values |
| --- | --- | --- | --- |
| **Site** | | | |
| `site_id` | random site id, stable across releases | 3174 |  |
| `viewport` | desktop (1041x750) or mobile (390x844) | 3174 | desktop, mobile |
| `platform` | coded ecommerce platform | 3174 | platform_A, platform_C, platform_P |
| `platform_moved` | the site no longer runs the platform whose reference list it came from | 3174 | False, True |
| `origin` | how the site entered the sample: reference_list (a platform's public customer list, every v1 row) or browsing (met while browsing, later releases) | 3174 | reference_list |
| `market` | market of the platform whose list the site came from: TR on every v1 row | 3174 | TR |
| `site_language` | language the homepage declares in its html lang attribute: tr, en or other; empty when not declared | 3152 | en, other, tr |
| `sector` | primary sector, project taxonomy (English label; Turkish source slugs under sector_tr in columns.json) | 3170 |  |
| **Banner** | | | |
| `banner_type` | none, info (notice only), cmp (a real choice), both | 3174 | both, cmp, info, none |
| `banner_forced` | the banner blocks the page until a choice is made (checked on every banner) | 1464 | False, True |
| `banner_position` | where the banner sits on the first screen | 1464 | bottom_bar, bottom_center, bottom_left, bottom_right, center_modal, fullscreen, side_panel, top_bar, top_box |
| `banner_area_share` | share of the first screen the banner covers, 0 to 1, rounded to 0.05 | 1464 |  |
| `backdrop` | the page behind the banner is dimmed | 1464 | False, True |
| `badge` | a persistent consent badge or icon is shown | 1464 | False, True |
| **Vendor** | | | |
| `vendor` | coded consent banner vendor; vendor_rare groups vendors on fewer than 5 sites | 1464 |  |
| `vendor_kind` | built_in (platform), third_party_cmp, library, other or unknown | 1464 | built_in, other, third_party_cmp, unknown |
| **Controls** | | | |
| `n_buttons` | buttons on the banner | 1464 | 0, 1, 2, 3, 4, 6 |
| `has_accept` | an accept button is present | 1464 | False, True |
| `reject` | how rejecting is offered: button, link or none | 1464 | button, link, none |
| `has_settings` | a settings or preferences control is present | 1464 | False, True |
| `has_close` | a close control is present | 1464 | False, True |
| **Colour** | | | |
| `site_theme` | page theme from the computed CSS background: light or dark | 1463 | dark, light |
| `site_temperature` | page background temperature: warm, cool or neutral | 1463 | cool, neutral, warm |
| `banner_theme` | banner background theme: light or dark | 1464 | dark, light |
| `banner_temperature` | banner background temperature | 1464 | cool, neutral, warm |
| `banner_hue_family` | banner background colour family | 1464 | black, blue, cyan, green, grey, orange, pink, purple, red, white, yellow |
| `accept_filled` | the accept button has a filled background | 1103 | False, True |
| `accept_temperature` | accept button colour temperature | 1103 | cool, neutral, warm |
| `accept_hue_family` | accept button colour family | 1103 | black, blue, cyan, green, grey, orange, pink, purple, red, white, yellow |
| `reject_filled` | the reject button has a filled background (button rejects only) | 478 | False, True |
| **Prominence** | | | |
| `accept_contrast_de` | CIEDE2000 band between the accept button's face (fill when filled, else border or label colour) and the banner background, dominant colours | 1103 | 0-5, 5-10, 10-20, 20-40, 40+ |
| `accept_luminance_ratio` | WCAG contrast ratio band of the accept button's face against the banner background | 1103 | <3, 3-4.5, 4.5-7, 7+ |
| `banner_page_de` | CIEDE2000 band between the banner background and the page as seen in the screenshot, dominant colours | 1462 | 0-5, 5-10, 10-20, 20-40, 40+ |
| `banner_ring_de` | CIEDE2000 band between the banner background and a 40 px ring of page right around it, dominant colours | 1462 | 0-5, 5-10, 10-20, 20-40, 40+ |
| `saliency_lift` | how much more the banner stands out than the page around it, rounded to 0.25 | 1462 |  |
| `nudge` | which of accept and reject is visually favoured: accept, equal or reject (a filled button over an outline one, else colour distance and area) | 925 | accept, equal, reject |
| **Accept against reject** | | | |
| `pair_same_colour` | accept and reject faces within CIEDE2000 10 (fill, or border or label colour for an outline button) | 477 | False, True |
| `pair_same_style` | accept and reject are both filled or both outline | 477 | False, True |
| `pair_font_weight_diff` | accept label font weight minus reject label font weight (page values) | 453 | -200, 0, 100, 200, 300 |
| `accept_label_contrast` | WCAG contrast ratio band of the accept label on its fill | 1103 | <3, 3-4.5, 4.5-7, 7+ |
| `reject_label_contrast` | WCAG contrast ratio band of the reject label on its fill | 478 | <3, 3-4.5, 4.5-7, 7+ |
| **Wording** | | | |
| `accept_wording` | accept_all, accept, acknowledge (ok, got it, close) or other | 1102 | accept, accept_all, acknowledge, other |
| `reject_wording` | reject_all, reject, necessary_only or other | 478 | necessary_only, other, reject, reject_all |
| `settings_wording` | customise, manage, settings, preferences or other | 853 | customise, manage, other, preferences, settings |

`columns.json` beside the file holds the same descriptions, the band edges, the sector
definitions and the Turkish source names of the sectors.

## Related work

Earlier studies of cookie banners measure EU or global sites. This dataset covers Turkish
ecommerce storefronts and adds per-banner layout and prominence measures from
human-labelled screenshots on desktop and mobile. Studies are listed whether or not they
released data, so a reader can see what has been measured before; what each released is
stated as found on its project or paper page in November 2026.

**Released data or tools**

- Bollinger, Kubicek, Cotrini and Basin, "Automating Cookie Consent and GDPR Violation
  Detection" (USENIX Security 2022): about 304k cookies with purpose labels from about
  30k websites, gathered from CMPs. Cookies rather than banner design. Data on Zenodo,
  https://doi.org/10.5281/zenodo.5838646
- Rasaii et al., BannerClick (PAM 2023, IMC 2023, PETS 2025): a tool built on OpenWPM that
  detects and clicks banners on top-ranked sites, used to study accept-or-pay banners and
  tracking after interaction. Code at https://github.com/bannerclick/bannerclick; data and
  scripts in the Max Planck Open Research Data Repository,
  https://doi.org/10.17617/3.1MUYFX, https://doi.org/10.17617/3.TREBZR,
  https://doi.org/10.17617/3.QZCILK
- Nouwens, Kristensen, Maalt and Bagge, "A Cross-Country Analysis of GDPR Cookie Banners
  and Flexible Methods for Scraping Them" (CHI 2025): the top 10,000 sites in each of 31
  countries (254,148 sites). The scraper is released as Consent Observatory (MPL-2.0),
  https://github.com/cavi-au/consent-observatory.eu; no dataset link found.
  https://arxiv.org/abs/2503.19655
- papayaverse, cookie_banner_dataset (2024, MIT): banner text and button labels of 561
  mostly global sites, buttons parsed by a language model, published with URLs and raw
  text. https://github.com/papayaverse/cookie_banner_dataset

**Studies (no dataset link found)**

- Utz, Degeling, Fahl, Schaub and Holz, "(Un)informed Consent: Studying GDPR Consent
  Notices in the Field" (CCS 2019): field experiments with more than 80,000 users on one
  German website; notice position, type of choice and nudging changed what users chose.
  https://arxiv.org/abs/1909.02638
- Matte, Bielova and Santos, "Do Cookie Banners Respect my Choice?" (2020): 1,426 sites
  with IAB Europe TCF banners out of 22,949 crawled in September 2019; suspected
  violations on 304 of the sites analysed. Browser extension source released (Cookie
  Glasses). https://www-sop.inria.fr/members/Nataliia.Bielova/cookiebanners/
- Nouwens, Liccardi, Veale, Karger and Kagal, "Dark Patterns after the GDPR" (CHI 2020):
  the designs of the five most popular CMPs on 680 of the top 10,000 UK sites; 11.8 % met
  minimal legal requirements. https://arxiv.org/abs/2001.02479
- Hils, Woods and Böhme, "Measuring the Emergence of Consent Management on the Web"
  (IMC 2020): 161 million browser crawls over 4.2 million domains; CMP adoption doubled
  from June 2018 to June 2019 and again by June 2020.
  https://informationsecurity.uibk.ac.at/pdfs/HWB2020_Consent_Management_IMC.pdf
- Bouhoula, Kubicek, Zac, Cotrini and Basin, "Automated Large-Scale Analysis of Cookie
  Notice Compliance" (USENIX Security 2024): 97k websites popular in the EU, crawled after
  reject, accept, save, close and no interaction.
- Uluk, "Sitemizi ziyaret ederek çerezleri kabul etmiş sayılırsınız" (Connectist 64,
  2023): the 50 most visited Turkish news sites in May 2022; all set non-necessary cookies
  without explicit consent. The closest Turkish study found; news sites, not ecommerce.
  https://doi.org/10.26650/CONNECTIST2023-1219698

No released dataset was found that covers Turkish ecommerce sites, labels banner layout
by hand, or measures both viewports.

## Use of AI tools

Sector classification used two local open-weight models, as described above. The
scanners, the build and analysis code and parts of this datasheet were written with AI
coding assistants (Claude Code, OpenAI Codex), which were also used to review the code
and the method. Banner labels, sector decisions where the models disagreed, and every
editorial decision were made by a person; all numbers here were recomputed from the
data.

## Limits

- A curated sample: reference stores of three platforms, not the platforms at large and
  not Turkish ecommerce at large.
- One visit per viewport on one day per site. A banner can change, be A/B tested or
  depend on location; this is what it looked like at capture time.
- Emulated viewports in a desktop browser, not real phones.
- One labeller, no second pass on a held-out set.
- The 3174 rows are 1587 sites seen at two viewports, not 3174 independent sites;
  analyses across viewports should treat the two rows of a site as paired.
- `nudge` and the prominence columns are rules applied to what is on screen; they were
  not tested on how visitors choose.
- An empty cell means the column does not apply on that row or could not be measured
  (for example no page record of a button's type size).
- Colour values from the page's own record exist only where that record matched the
  screenshot; elsewhere the label colour comes from the screenshot. Some font values
  come from a later visit than the screenshot.

## Versions

| Version | Date | Change |
| --- | --- | --- |
| v1.0.0 | 2026-11-09 | first release |
| v1.0.1 | 2026-11-14 | label fix: on two sites (4 rows, both viewports) of one vendor the accept button was labelled as an other button; `has_accept`, the `accept_*` columns, `nudge` and the `pair_*` columns change on those rows. Datasheet: related work section |

Site ids and codes stay the same across releases.

## Files

- `consent-sites.csv`: the data, UTF-8, one header row.
- `columns.json`: column descriptions, band edges, the sector definitions and the
  Turkish source names of the sectors.
- `SHA256SUMS`: checksums of the two files above.

## Licence and citation

The data and this datasheet are licensed under CC BY 4.0 (see `LICENSE`): free to use,
share and adapt, including commercially, with attribution. Attribution names the author
and the dataset with its version and links to the dataset:

> Ceyhun Aksan, "Consent banners on ecommerce storefronts", v1.0.1,
> https://doi.org/10.5281/zenodo.24090344 (CC BY 4.0)

`CITATION.cff` holds the same for citation tools, `.zenodo.json` for Zenodo.
