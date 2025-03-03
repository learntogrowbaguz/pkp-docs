# PKP Documentation Hub

The repository for generating PKP's documentation hub.

Interested in contributing documentation? Check out [our guidelines for contributing to PKP documentation](https://docs.pkp.sfu.ca/contributing/) to get started.

## Community Code of Conduct

This repository is one of PKP's community spaces and all activities here are guided by [PKP's Code of Conduct](https://pkp.sfu.ca/code-of-conduct/). Please review the Code and help us create a welcoming environment for all participants.

## Usage

[Install Ruby 2.1.0 or higher](https://www.ruby-lang.org/en/documentation/installation/) and install the bundler gem.

```
gem install bundler
```

Access the `pkp-docs` root folder and install Jekyll to build or serve the site.

```
bundle install
```

Serve the site locally.

```
bundle exec jekyll serve
```

## Orientation to file structure

The index page's data is specified in `_data/index.yml`. Each app covered is specified under the `apps` property. OJS 3 looks like this:

```
- path: ojs3
	title: Open Journal Systems 3
	titleShort: OJS
	includeInHero: true
	description: Our flagship software for open access journal publishing, used by more than 10,000 journals worldwide.
	cards:
		- path: learning-ojs-3
		- path: looking-for-ojs-2
			isHighlighted: true
		- path: developer
			isDeveloper: true
		- path: starting-a-journal
```

- `path` should be a unique ID made up of numbers and letters with no spaces.
- `title` is the main title of the app's section
- `titleShort` is used in the buttons in the hero element at the top
- `includeInHero` can be set to false if you want to create an app section but don't want it to be listed in the buttons in the hero element at the top
- `description` is a short description that will be placed below the title in the app's section
- `cards` see below

Each of the `cards` provides a `path` which points to markdown files in the `_includes/cards` directory. Each of these cards represents a resource card that will be added under that app, and you can use markdown to add links. See the next section.

### Add a new resource

Create a card under `_includes/cards/<app>/<your-new-resource-name>.md`. Give your card a title with three `###`:

```
### [Your resource title](http://link-to-resource.com)
```

Add a short description below it, and link to the resource.

```
This resource will help you accomplish X. [View Now](http://link-to-resource.com)
```

### Create multiple sections in your resource card

If you want to add a section to the card, use three dashes before the section:

```
---
- [Table](/url)
- [Of](/url)
- [Contents](/url)
---
Available in [Deutsch](/url).
```

## Versions of documents

You can link between different versions of the same document. Below is an example using the `importing-exporting` document. First, prepare all versions of the document in directories like below. (Notice the latest version has no version number.)

```
/importing-exporting/en
/importing-exporting/2.0/en
/importing-exporting/1.0/en
```

Then add an entry for the version in `/data/versions.yaml`:

```
importing-exporting:
	3.0: /importing-exporting/en
	2.0: /importing-exporting/2.0/en
	1.0: /importing-exporting/1.0/en
```

Then add the following to the frontmatter of each `.md` file in the document, adjusting the `version` entry as appropriate:

```
---
book: importing-exporting
version: 2.0
---
```

This should display a link to each version in the header of each `.md` file you've modified.

### Creating a new version

To create a new version of the document:

1. Move the existing `/importing-exporting/en` to `/importing-exporting/<version>/en`.
2. Move the new version into `/importing-exporting/en`.
3. Update `/_data/versions.yaml` appropriately.

The base URL, `/<any-book>/en`, should always be the current version so that any internal or external links always point to the most up-to-date resource.

### Multiple languages

This structure doesn't yet support versioning separate language editions of documents. We can work on that when we need it.

## Block search indexing

To block indexing in search engines, add the following to the frontmatter in each `.md` file in a document:

```
---
noindex: true
---
```

This will add `<meta name="robots" content="noindex">` to each page.

## Generate REST API References

The REST API references use [redoc](https://github.com/Redocly/redoc) to generate the human-readable documentation from an OpenAPI json file. To build the REST API references you will need a checkout of the application for the version you wish to generate a reference.

Create the JSON file from the application and place it anywhere.

```
php lib/pkp/tools/buildSwagger.php ~/3.3.json
```

Install [redoc-cli](https://www.npmjs.com/package/redoc-cli) globally if you don't have it.

```
npm install -g redoc-cli
```

From the root directory of the the docs repository, run the following command to build the docs.

```
redoc-cli bundle ~/3.3.json --options=.redoc.json --output=dev/api/ojs/3.3.html
```

Then add a link to the new file in the [API Guide](./dev/api/index.md) and the [card](./_includes/cards/dev/rest-api.md).

## Generate Database References

The database references are generated with [SchemaSpy](https://schemaspy.org/). To generate a reference, follow the [installation instructions](https://schemaspy.readthedocs.io/en/latest/installation.html) and then run SchemaSpy with the [configuration](https://schemaspy.readthedocs.io/en/latest/started.html) parameters to point to the database you want to profile.

Once the documentation is generated, move the whole directory into the docs hub. For example, if the documentation is for OJS 3.5, run the following command to move it to the correct directory in the docs hub:

```
mv <schema-spy-output-dir> <docs-hub>/dev/database/ojs/3.5
```

Add a link to the documentation from `/dev/database/index.md`.
