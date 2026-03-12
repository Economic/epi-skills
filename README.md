# epi-skills

A Claude Code plugin marketplace for economics and data analysis. We use these skills at the Economic Policy Institute.

## Available plugins

| Plugin | Description |
|--------|-------------|
| [r-econ-data](plugins/r-econ-data/) | Economic data sources and retrieval methods for R (EPI SWADL, BLS, BEA, FRED, Census, IPUMS, CPS microdata). Includes skills for retrieving data, calculating statistics, and validating analyses. |
| [research](plugins/research/) | Research skills and workflows, including techniques for retrieving and investigating remote codebases and documentation. |

## Installation

Add the marketplace:

```
/plugin marketplace add economic/epi-skills
```

Then install a plugin:

```
/plugin install r-econ-data@epi-skills
/plugin install research@epi-skills
```

## Adding a new plugin

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to add a plugin to this marketplace.
