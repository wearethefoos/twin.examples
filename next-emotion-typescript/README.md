# Next + Emotion + TypeScript

Download this example using [degit](https://github.com/Rich-Harris/degit):

```shell
npx degit https://github.com/ben-rogerson/twin.examples/next-emotion-typescript folder-name
```

## Getting started

### Installation

Install Next.js

```shell
npx create-next-app --typescript
```

Install the dependencies

```shell
npm install @emotion/react @emotion/styled @emotion/css @emotion/server
npm install -D twin.macro tailwindcss babel-plugin-macros @babel/preset-react @babel/preset-typescript @emotion/babel-plugin babel-loader
```

<details>
  <summary>Install with Yarn</summary>

```shell
yarn create next-app --typescript
```

Install the dependencies

```shell
yarn add @emotion/react @emotion/styled @emotion/css @emotion/server
yarn add -D twin.macro tailwindcss babel-plugin-macros @babel/preset-react @babel/preset-typescript @emotion/babel-plugin babel-loader
```

</details>

### Add the global styles

Twin uses the same [preflight base styles](https://unpkg.com/tailwindcss/dist/base.css) as Tailwind to smooth over cross-browser inconsistencies.

The `GlobalStyles` import adds these base styles along with some @keyframes for the animation classes and some global css that makes the [ring classes](https://tailwindcss.com/docs/ring-width) and box-shadows work.

You can add Twin’s `GlobalStyles` import in `styles/GlobalStyles.tsx`:

```ts
// styles/globalStyles.tsx
import React from 'react'
import { Global } from '@emotion/react'
import tw, { css, theme, GlobalStyles as BaseStyles } from 'twin.macro'

const customStyles = css({
  body: {
    WebkitTapHighlightColor: theme`colors.purple.500`,
    ...tw`antialiased`,
  },
})

const GlobalStyles = () => (
  <>
    <BaseStyles />
    <Global styles={customStyles} />
  </>
)

export default GlobalStyles
```

Then import the global styles in `pages/_app.tsx`:

```ts
// pages/_app.tsx
import { cache } from '@emotion/css'
import { CacheProvider } from '@emotion/react'
import type { AppProps } from 'next/app'
import GlobalStyles from './../styles/GlobalStyles'

const App = ({ Component, pageProps }: AppProps) => (
  <CacheProvider value={cache}>
    <GlobalStyles />
    <Component {...pageProps} />
  </CacheProvider>
)

export default App
```

### Add the twin config

Twin’s config can be added in a couple of different files.

a) Either in `babel-plugin-macros.config.js`:

```js
// babel-plugin-macros.config.js
module.exports = {
  twin: {
    preset: 'emotion',
  },
}
```

b) Or in `package.json`:

```js
// package.json
"babelMacros": {
  "twin": {
    "preset": "emotion"
  }
},
```

### Add the babel config

Add this babel configuration in `.babelrc.js`:

```js
// In .babelrc.js
module.exports = {
  presets: [
    [
      'next/babel',
      {
        'preset-react': {
          runtime: 'automatic',
          importSource: '@emotion/react',
        },
      },
    ],
  ],
  plugins: ['@emotion/babel-plugin', 'babel-plugin-macros'],
}
```

### Add the server stylesheet

To avoid the ugly Flash Of Unstyled Content (FOUC), add a server stylesheet in `pages/_document.tsx` that gets read by Next.js:

```js
// pages/_document.tsx
import React from 'react'
import Document, { Html, Head, Main, NextScript } from 'next/document'
import { extractCritical } from '@emotion/server'

export default class MyDocument extends Document {
  static async getInitialProps(ctx: any) {
    const initialProps = await Document.getInitialProps(ctx)
    const critical = extractCritical(initialProps.html)
    initialProps.html = critical.html
    initialProps.styles = (
      <React.Fragment>
        {initialProps.styles}
        <style
          data-emotion-css={critical.ids.join(' ')}
          dangerouslySetInnerHTML={{ __html: critical.css }}
        />
      </React.Fragment>
    )

    return initialProps
  }

  render() {
    return (
      <Html lang="en">
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}
```

### Complete the TypeScript setup

Because twin routes the `styled` and `css`, you’ll need complete the typescript setup.

Create a `types/twin.d.ts` file in your project root and add these declarations:

```typescript
// types/twin.d.ts
import "twin.macro";
import { css as cssImport } from "@emotion/react";
import styledImport from "@emotion/styled";
import { CSSInterpolation } from "@emotion/serialize";

declare module "twin.macro" {
  // The styled and css imports
  const styled: typeof styledImport;
  const css: typeof cssImport;
}

declare module "react" {
  // The tw and css prop
  interface DOMAttributes<T> {
    tw?: string;
    css?: CSSInterpolation;
  }
}
```

Then add the following in your typescript config:

```typescript
// tsconfig.json
{
  // Tell typescript about the types folder
  "types": ["types"],
  // Recommended settings
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve"
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"],
}
```

[](#usage)

## Usage

### Styled components

Use the tw import to create and style new components:

```ts
import tw from 'twin.macro'

const Input = tw.input`border hover:border-black`

;<Input />
```

Switch to the styled import to add conditional styling:

```ts
import tw, { styled } from 'twin.macro'

const StyledInput = styled.input({
  // Spread the base styles
  ...tw`bg-white max-w-[200px]`,
  // Add conditional styling in the variants object
  // https://stitches.dev/docs/variants
  variants: {
    hasBorder: { true: tw`border-purple-500` },
  },
})

;<StyledInput hasBorder />
```

### Prop styling

Style jsx elements using the tw prop:

```ts
import 'twin.macro'

const Input = () => <input tw="border hover:border-black" />
```

Nest Twin’s `tw` import within a css prop to add conditional styles:

```ts
import tw from 'twin.macro'

const Input = ({ hasHover }) => (
  <input
    css={{
      // Spread the base styles
      ...tw`border`,
      // Add conditionals afterwards
      ...(hasHover && tw`hover:border-black`),
    }}
  />
)
```

Or mix sass styles with the css import:

```ts
import tw, { css } from 'twin.macro'

const hoverStyles = {
  '&:hover': {
    'border-color': 'black',
    ...tw`text-black`,
  },
}

const Input = ({ hasHover }) => (
  <input css={{ ...tw`border`, ...(hasHover && hoverStyles) }} />
)
```

[](#customization)

## Customization

- [View the config options →](https://github.com/ben-rogerson/twin.macro/blob/master/docs/options.md)
- [Customizing the tailwind config →](https://github.com/ben-rogerson/twin.macro/blob/master/docs/customizing-config.md)

[](#next-steps)

## Next steps

Learn how to work with twin

- [The prop styling guide](https://github.com/ben-rogerson/twin.macro/blob/master/docs/prop-styling-guide.md) - A must-read guide to level up on prop styling
- [The styled component guide](https://github.com/ben-rogerson/twin.macro/blob/master/docs/styled-component-guide.md) - A must-read guide on getting productive with styled-components

Learn more about emotion

- [Emotion’s css prop](https://emotion.sh/docs/css-prop)
- [Emotion’s css import](https://emotion.sh/docs/css-prop#string-styles)
- [Emotion’s styled import](https://emotion.sh/docs/styled)
