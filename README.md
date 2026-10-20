# PageLoader

Download a web page and its local resources from the command line.

[![Node CI](https://github.com/luisfelipemontoya/PageLoader/actions/workflows/nodejs.yml/badge.svg)](https://github.com/luisfelipemontoya/PageLoader/actions/workflows/nodejs.yml)
[![Hexlet checks](https://github.com/luisfelipemontoya/PageLoader/actions/workflows/hexlet-check.yml/badge.svg)](https://github.com/luisfelipemontoya/PageLoader/actions/workflows/hexlet-check.yml)

**PageLoader** is a Node.js utility that saves a page's HTML alongside images, stylesheets, scripts, and other linked resources hosted on the same hostname. It rewrites the corresponding HTML references to point to the downloaded files.

Use it to inspect saved HTML, collect static page assets, or explore asynchronous file and network operations in a modular CLI project.

## Features

- Download a page using the `page-loader` command.
- Collect same-hostname resources referenced by `img[src]`, `link[href]`, and `script[src]`.
- Resolve relative resource URLs against the page URL.
- Rewrite downloaded resource references to local paths.
- Download resources concurrently with task progress reporting.
- Enable diagnostic logs through the `DEBUG` environment variable.
- Use the downloader programmatically through a Promise-based API.

## Installation

Prerequisites: Git, Node.js 20 or later, and npm.

Install from source:

```bash
git clone https://github.com/luisfelipemontoya/PageLoader.git
cd PageLoader
npm ci
npm link
```

Verify that the command is available:

```bash
page-loader --help
```

To run directly from the repository without `npm link`:

```bash
node bin/page-loader.js --help
```

## Quick start

Create an output directory and download a page:

```bash
mkdir -p downloads
page-loader --output ./downloads https://example.com
```

After a successful download, the CLI prints the saved HTML file path:

```text
downloads/example-com.html
```

The resulting structure is:

```text
downloads/
├── example-com.html
└── example-com_files/
```

The resource directory contains any matching assets found in the HTML. It is created even when the page has no matching resources.

Open the saved HTML in a browser to inspect the result. Its offline appearance depends on the resources and behavior the page requires.

## CLI reference

```text
page-loader [options] <url>
```

| Argument or option | Description |
| --- | --- |
| `<url>` | Absolute URL of the page to download. |
| `-o, --output [dir]` | Destination directory. Defaults to the current working directory. |
| `-h, --help` | Show usage information. |
| `-V, --version` | Show the CLI version. |

The destination directory must already exist and be writable. When using `--output`, supply a directory value explicitly.

Download into the current directory:

```bash
page-loader https://example.com
```

Download into a specific directory:

```bash
page-loader --output ./downloads https://example.com
```

The CLI exits with status `1` and prints an error message if the page, a resource, or a required filesystem operation fails.

## How downloads work

1. Fetch the page HTML.
2. Find resource URLs in `img[src]`, `link[href]`, and `script[src]`.
3. Keep resources whose hostname matches the requested page's hostname.
4. Create a resource directory and download the collected files concurrently.
5. Rewrite the matching HTML attributes and save the updated document.

For a page at `https://example.com/blog`, the generated names are:

```text
example-com-blog.html
example-com-blog_files/
```

An image at `https://example.com/assets/logo.png` is saved inside that resource directory as `example-com-assets-logo.png`.

### Scope and limitations

- Resources on other hostnames retain their original remote URLs.
- Only references in the downloaded HTML are collected. Resources referenced inside CSS or JavaScript are not downloaded recursively.
- JavaScript is not executed, so content and assets loaded dynamically are not captured.
- Page links in `<a>` elements are not followed.
- Query strings and URL fragments are not included in generated filenames, so URLs that differ only in those components can map to the same filename.
- The generated resource directory must not already exist. Use a fresh output directory when downloading the same page again.
- A failed download can leave partially downloaded resources in the output directory.

## Debugging

Enable diagnostic logs to inspect page fetching, resource discovery, resource downloads, and HTML saving:

```bash
DEBUG=page-loader page-loader --output ./downloads https://example.com
```

The environment variable syntax above applies to Bash and similar shells.

## Programmatic usage

From an ES module in the repository root:

```javascript
import pageLoader from './index.js';

try {
  const filePath = await pageLoader('https://example.com', './downloads');
  console.log(`Saved page: ${filePath}`);
} catch (error) {
  console.error(error.message);
}
```

`pageLoader(url, outputDir)` returns a Promise that resolves to the saved HTML file path. The output directory defaults to `process.cwd()` and must already exist.

## Demonstrations

Recorded walkthroughs:

- [Basic usage and image downloads](https://asciinema.org/a/AbkKWN2czJJG5mmK)
- [Downloading linked resources and scripts](https://asciinema.org/a/7LFUeq4YWDJxU05Q)
- [Debug logs](https://asciinema.org/a/1svlR7997zsSAX3y)
- [Error handling](https://asciinema.org/a/HtZvS75AWSh4MYTb)
- [Download progress](https://asciinema.org/a/21TCwpNnlXSnc5l9)

## Development

Install dependencies with `npm ci`, then run the project checks:

```bash
npm run check
```

| Command | Purpose |
| --- | --- |
| `npm test` | Run the Jest test suite. |
| `npm run test:ci` | Run tests with coverage reporting. |
| `npm run lint` | Check code with ESLint. |
| `npm run lint:fix` | Apply automatic ESLint fixes. |
| `npm run format:check` | Check formatting with Prettier. |
| `npm run format` | Format project files with Prettier. |
| `npm run check` | Run formatting checks, linting, and tests. |

The tests use Nock to mock HTTP responses and temporary directories for downloaded files. They cover HTML downloads, image downloads, CSS and JavaScript resources, external resource exclusion, and selected failure cases.

### Project structure

| Path | Purpose |
| --- | --- |
| `bin/page-loader.js` | CLI arguments, output reporting, and exit status. |
| `src/pageLoader.js` | Page fetching, resource collection, downloads, and HTML rewriting. |
| `index.js` | Public module entry point. |
| `__tests__/` | Jest tests with mocked HTTP requests. |
| `__fixtures__/` | Supporting test fixtures. |
| `.github/workflows/` | Continuous integration workflows. |

### Built with

- **Axios** for HTTP requests.
- **Cheerio** for HTML parsing and rewriting.
- **Commander.js** for the CLI.
- **Listr2** for concurrent download tasks and progress.
- **debug** for diagnostic logging.
- **Jest and Nock** for automated testing.

## Contributing

Report bugs or propose improvements through [issues](https://github.com/luisfelipemontoya/PageLoader/issues) and pull requests. Include the command, page URL, and relevant error output when reporting a problem. Run `npm run check` before submitting changes.

## License

The package declares the ISC license in [package.json](package.json).

