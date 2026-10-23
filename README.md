# elite-shared

Shared files for Elite Contract Furniture's internal apps. No app code lives here.

| Path | What it is |
|---|---|
| [HANDOVER.md](HANDOVER.md) | Handover note: apps, priorities, decisions made, open items. Read it first; update it at the end of every piece of work. |
| `design/` | Shared design file: Elite brand colours, fonts, buttons, forms, logos. Every app takes its styles from here. |
| `docs/` | Architecture and design notes that cover more than one app, such as the survey to FileMaker data flow. |

App repos:
* [EliteFM](https://github.com/donniemurdo/EliteFM) (FileMaker, docs only)
* [elite-softfurnishing-survey](https://github.com/donniemurdo/elite-softfurnishing-survey) (soft furnishings survey, admin page, the Cloudflare Worker)
* [elite-bed-survey](https://github.com/donniemurdo/elite-bed-survey) (bed survey)

Each app keeps its own copy of the brand CSS and fonts so it works offline. When the design here changes, copy it into the apps.
