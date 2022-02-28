![jsx-pdf logo](https://user-images.githubusercontent.com/11426386/43649419-0819a78a-9735-11e8-8a74-12d26e5e830b.jpg)

[![npm version](https://badge.fury.io/js/jsx-pdf.svg)](https://www.npmjs.com/jsx-pdf) [![Build Status](https://travis-ci.org/schibsted/jsx-pdf.svg?branch=master)](https://travis-ci.org/schibsted/jsx-pdf) [![Coverage Status](https://coveralls.io/repos/github/schibsted/jsx-pdf/badge.svg?branch=master)](https://coveralls.io/github/schibsted/jsx-pdf?branch=master)

Generate modular PDFs via [pdfmake](http://pdfmake.org/) using JSX.

```jsx
import PDFMake from 'pdfmake';
import JsxPdf from 'jsx-pdf';
import { OpenSans } from './font-descriptors';

const pdfMake = new PDFMake({
  OpenSans,
});

const stream = pdfMake.createPdfKitDocument(
  JsxPdf.renderPdf(
    <document defaultStyle={{ font: 'OpenSans', fontSize: 12 }}>
      <content>This will appear in my PDF!</content>
    </document>,
  ),
);

// write the stream to a file; this could also be streamed to an HTTP connection, stdout etc
stream.on('end', () => console.log('PDF generated'));
stream.pipe(fs.createWriteStream('~/Desktop/test.pdf'));
stream.end();
```

## Quick start

### Javascript

- `yarn add jsx-pdf @babel/plugin-transform-react-jsx`
- Configure the part of your build that transpiles your JSX to use `JsxPdf.createElement` as the element factory.
  - For babel, add the configuration below to your `.babelrc`.
  ```json
  "plugins": [
    [
      "@babel/plugin-transform-react-jsx",
      { "pragma": "JsxPdf.createElement", "pragmaFrag": "JsxPdf.Fragment" }
    ]
  ]
  ```

### Typescript

- `yarn add -D @types/jsx-pdf`
- For TypeScript, add the configuration below to your `tsconfig.json`. Setting `jsx` to `react` configures TypeScript to handle JSX transpiling for you, and the `jsxFactory` option specifies the element factory to use. Setting `jsxFragmentFactory` allows you to use [JSX Fragments](https://reactjs.org/docs/fragments.html#short-syntax).

```json
"compilerOptions": {
  "jsx": "react",
  "jsxFactory": "JsxPdf.createElement",
  "jsxFragmentFactory": "JsxPdf.Fragment",
},
```

- Code away! See the examples below.

You can also run our example script by running `yarn demo` and opening the generated pdf at `example/example.pdf`. Check the console logs for additional information.

## Components

Similar to modern front-end frameworks, you can define your own components using declarative syntax.

#### Basic example:

```jsx
import JsxPdf from 'jsx-pdf';

const Greeting = ({ name }) => <text>Hello, {name}!</text>;

const doc = (
  <document>
    <content>
      <Greeting name="Bob" />
    </content>
  </document>
);
```

#### List example:

```jsx
import JsxPdf from 'jsx-pdf';

const GroupGreeting = ({ names }) => (
  <>
    {names.map((name) => (
      <Greeting name={name} />
    ))}
  </>
);

const doc = (
  <document>
    <content>
      <GroupGreeting names={['Bob', 'Alice']} />
    </content>
  </document>
);
```

#### Inline If example:

```jsx
import JsxPdf from 'jsx-pdf';

const Signature = () => <text>JSX-PDF, Inc.</text>;

const SignedGreeting = ({ name }) => (
  <>
    {name && <Greeting name={name} />}
    <Signature />
  </>
);

const doc = (
  <document>
    <content>
      <SignedGreeting />
    </content>
  </document>
);
```

#### Inline If-Else example:

```jsx
import JsxPdf from 'jsx-pdf';

const AnonymousGreeting = () => <text>We don't know you.</text>;

const SignedGreeting = ({ name }) => (
  <>
    {name ? <Greeting name={name} /> : <AnonymousGreeting />}
    <Signature />
  </>
);

const doc = (
  <document>
    <content>
      <SignedGreeting />
    </content>
  </document>
);
```

#### Element variable example:

```jsx
import JsxPdf from 'jsx-pdf';

const SignedGreeting = ({ name }) => {
  let greeting;

  if (name) {
    greeting = <Greeting name={name} />;
  } else {
    greeting = <AnonymousGreeting />;
  }

  return (
    <>
      {greeting}
      <Signature />
    </>
  );
};

const doc = (
  <document>
    <content>
      <SignedGreeting />
    </content>
  </document>
);
```

### Styling

Styling can be done by adding appropriate attributes to tags. It's often helpful for readability to group style-related attributes together and use the spread syntax.

```jsx
import JsxPdf from 'jsx-pdf';

const Greeting = ({ name }) => {
  const styles = {
    italics: true,
    fontSize: 10,
  };

  return <text {...styles}>Hello, {name}!</text>;
};

const doc = (
  <document>
    <content>
      <Greeting name="Bob" />
    </content>
  </document>
);
```

### Context

Each component has access to global context and can update it if necessary.

```jsx
import JsxPdf from 'jsx-pdf';

const AllowedUsersProvider = (attributes, context, updateContext) => {
  updateContext({
    allowedUsers: ['Alice'],
  });

  return attributes.children[0];
};

const SecureGreeting = ({ name }, { allowedUsers }) =>
  allowedUsers.includes(name) ? (
    <text>Hello, {name}!</text>
  ) : (
    <text>You are not allowed.</text>
  );

const doc = (
  <AllowedUsersProvider>
    <document>
      <content>
        <SecureGreeting name="Bob" />
      </content>
    </document>
  </AllowedUsersProvider>
);
```

## Document primitives

This section describes basic elements provided by the library. More information about supported attributes and advanced examples can be found [here](http://pdfmake.org/playground.html).

### Top elements

Each document has to be enclosed within `document` tag with nested `content`, and optional `header` and `footer`. The document is the place for configuration that affects the whole PDF, such as page margins, page size, default style, and metadata.

```jsx
import JsxPdf from 'jsx-pdf';

const doc = (
  <document
    pageMargins={[20, 20, 20, 20]}
    pageSize="A4"
    defaultStyle={{
      font: 'OpenSans',
    }}
    info={{
      author: 'Buzz Lightyear',
    }}
  >
    <header>Greeting</header>
    <content>Hello, Bob!</content>
    <footer>JSX-PDF, Inc.</footer>
  </document>
);
```

### Dynamic Header and Footer

If you want to use the [dynamic header functionality](https://pdfmake.github.io/docs/document-definition-object/headers-footers/) in pdfmake, simply pass a render function as the only child of the header or footer:

```jsx
const doc = (
  <document>
    <header>
      {(currentPage, pageCount) => (
        <text>
          Page {currentPage} of {pageCount}.
        </text>
      )}
    </header>
    <content>{/* ... */}</content>
  </document>
);
```

The parameters are:

- `currentPage` - the 1-indexed page for which the content is being rendered
- `pageCount` - the total number of pages in the document
- `pageSize` - an object containing information about the dimensions of the page.

### Paragraphs

Paragraphs are defined using `text` tag.

<!-- prettier-ignore -->
```jsx
import JsxPdf from 'jsx-pdf';

const doc = (
  <document>
    <content>
      <text>
        This sentence will be rendered as one paragraph,

        even though there are

        line


        breaks.
      </text>
      <text>This is another paragraph.</text>
    </content>
  </document>
);
```

In order to apply styling to a group of paragraphs, they can be wrapped with a `stack` tag.

```jsx
import JsxPdf from 'jsx-pdf';

const doc = (
  <document>
    <content>
      <stack color="red">
        <text>First red paragraph.</text>
        <text>Second red paragraph.</text>
      </stack>
      <text color="blue">Blue parahraph.</text>
    </content>
  </document>
);
```

### Columns

Elements nested in `columns` tag will be stacked horizontally.

```jsx
import JsxPdf from 'jsx-pdf';

const doc = (
  <document>
    <content>
      <columns columnGap={10}>
        <column width={100}>Fixed width column</column>
        <column width="10%">Percentage width column</column>
        <column width="auto">
          Column that adjusts width based on the content
        </column>
        <column width="*">Column that fills the remaining space</column>
      </columns>
    </content>
  </document>
);
```

### Lists

Both ordered and unordered lists are supported.

```jsx
import JsxPdf from 'jsx-pdf';

const docWithOrderedList = (
  <document>
    <content>
      <ol reversed start={10} separator={['(', ')']} type="lower-roman">
        <text>Item 1</text>
        <text>Item 2</text>
        <text>Item 3</text>
      </ol>
    </content>
  </document>
);

const docWithUnorderedList = (
  <document>
    <content>
      <ul color="blue" markerColor="red" type="square">
        <text>Item 1</text>
        <text>Item 2</text>
        <text>Item 3</text>
      </ul>
    </content>
  </document>
);
```

### Tables

`table` tag provides a simple way of creating table layouts.

```jsx
const leftCellStyle = {
  color: 'grey',
};

const doc = (
  <document>
    <content>
      <table widths={[100, '*', 'auto']} headerRows={1} layout="headerLineOnly">
        <row>
          <cell>Fixed width column</cell>
          <cell>Column that fills the remaining space</cell>
          <cell>Column that adjusts width based on the content</cell>
        </row>
        <row>
          <cell {...leftCellStyle}>Cell 1.1</cell>
          <cell>Cell 1.2</cell>
          <cell>Cell 1.3</cell>
        </row>
        <row>
          <cell {...leftCellStyle}>Cell 2.1</cell>
          <cell>Cell 2.2</cell>
          <cell>Cell 2.3</cell>
        </row>
      </table>
    </content>
  </document>
);
```

### Images

`image` supports JPEG and PNG formats.

```jsx
import JsxPdf from 'jsx-pdf';

const doc = (
  <document>
    <content>
      <image src="/home/bob/photos/Bob.png" width={150} height={150} />
    </content>
  </document>
);
```

### SVGs

The `svg` tag can be used to render SVG images. The `width`, `height` and `fill` attributes can be used to control the size of the image as described in the [pdfmake docs](https://pdfmake.github.io/docs/document-definition-object/svgs/).

```jsx
import JsxPdf from 'jsx-pdf';

const doc = (
  <document>
    <content>
      <svg
        content={`
          <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
            <circle fill="red" cx="50" cy="50" r="50"/>
          </svg>
        `}
      />
    </content>
  </document>
);
```

### QR Codes

The `qr` tag can be used to render QR codes. There are various options available as described in the [pdfmake docs](https://pdfmake.github.io/docs/0.1/document-definition-object/qr/).

```jsx
import JsxPdf from 'jsx-pdf';

const doc = (
  <document>
    <content>
      <qr content="My text" />
    </content>
  </document>
);
```

## API

### renderPdf

Accepts JSX and returns a PDF JSON representation in the format expected by pdfmake.

### createElement

This function converts JSX to object representation. Every time JSX syntax is used, the function has to be made available. The functionality depends on the babel plugin `@babel/plugin-transform-react-jsx` (or equivalent), and Babel must be set up in the project in order to transpile the JSX correctly.

Example `.babelrc` file:

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "node": "current"
        }
      }
    ]
  ],
  "plugins": [
    [
      "@babel/plugin-transform-react-jsx",
      { "pragma": "JsxPdf.createElement", "pragmaFrag": "JsxPdf.Fragment" }
    ]
  ]
}
```

## Disclaimer

Copyright 2018 Schibsted

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## License

By contributing to this project, you agree that your contributions will be licensed under its MIT license.
