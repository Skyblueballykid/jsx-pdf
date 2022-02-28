# 2.3.1

## Other changes

- Updated dependencies

# 2.3.0

## New Features

- JSX Fragments are now supported in place of `<stack>`. Babel configuration will need updating to use this as described in the README.md
- QR codes are now supported `<qr content="https://example.org" />`

# 2.2.0

## New Features

- SVGs are now supported

## Other changes

- All dependencies updated to latest version. No breaking changes affecting consumers.

# 2.1.1

- All dependencies updated to latest version. This means that we've switched from `babel-plugin-transform-react-jsx` to `@babel/plugin-transform-react-jsx` in our peerDependencies and documentation. While this means that if you're still using the old package you'll get a warning, both versions of the plugin should work just fine.

# 2.1.0

## New Features

- It's now possible to obtain the page, page count, and page size in the header and footer by passing a render function as the only child. See the readme for an example.

# 2.0.0

## Breaking changes

- Switches to a single default export - before, it was necessary to configure babel to use `createElement`, and import the export of the same name from jsx-pdf. Now, babel should be configured to use `JsxPdf.createElement`, and the whole package should be imported. See the readme for more details and examples.
- Changes the integration point of the package. Previously pdfmake was a dependency, and the library handled creating the PDFKit document internally. However, this limits the ability of consumers to configure pdfmake in the way that they need to. We also only supported rendering PDFs in node, but pdfmake can actually render PDFs in a browser too. Now, we provide a renderPdf function that outputs the correct pdfmake format, and consumers need to handle importing pdfmake and passing the result of renderPdf to it.

# 1.0.0

- First semver release 🎉
