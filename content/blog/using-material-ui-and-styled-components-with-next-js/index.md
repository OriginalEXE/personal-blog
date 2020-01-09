---
title: Using Material UI and Styled Components with Next.js
date: "2019-05-25T11:20:23.000Z"
description: "How to combine Material UI, Styled Components and Next.js"
---

With the release of Material UI 4.0, I decided to give it a shot in one of my projects. I was already using Styled Components there which I was server-side rendering by customizing the Next.js's `_document.js` file.

The problem was that Material UI uses JSS and so I needed to add its server-side rendering as well. After quite some time and trying a few different things, I have realized that the solution is actually quite simple, here is what you will need:

`_document.js`:

```js
/* eslint-disable react/jsx-filename-extension */
import React from "react"
import Document, { Head, Main, NextScript } from "next/document"
import { ServerStyleSheets } from "@material-ui/styles"
import { ServerStyleSheet } from "styled-components"
import flush from "styled-jsx/server"

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const sheet = new ServerStyleSheet()
    const sheets = new ServerStyleSheets()
    const originalRenderPage = ctx.renderPage

    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: App => props =>
            sheet.collectStyles(sheets.collect(<App {...props} />)),
        })

      const initialProps = await Document.getInitialProps(ctx)

      return {
        ...initialProps,
        styles: (
          <>
            {initialProps.styles}
            {sheets.getStyleElement()}
            {sheet.getStyleElement()}
            {flush() || null}
          </>
        ),
      }
    } finally {
      sheet.seal()
    }
  }

  render() {
    return (
      <html lang="en">
        <body>
          <Main />
          <NextScript />
        </body>
      </html>
    )
  }
}
```

`_app.js`:

```js
/* eslint-disable react/jsx-filename-extension */
import React from "react"
import App, { Container } from "next/app"
import Head from "next/head"
import { ThemeProvider } from "@material-ui/styles"
import CssBaseline from "@material-ui/core/CssBaseline"
import { GlobalStyle } from "../css/global"
import theme from "../css/theme"

export default class MyApp extends App {
  componentDidMount() {
    const jssStyles = document.querySelector("#jss-server-side")

    if (jssStyles) {
      jssStyles.parentNode.removeChild(jssStyles)
    }
  }

  render() {
    const { Component, pageProps } = this.props

    return (
      <Container>
        <React.StrictMode>
          <ThemeProvider theme={theme}>
            <CssBaseline />
            <GlobalStyle />
            <Component {...pageProps} />
          </ThemeProvider>
        </React.StrictMode>
      </Container>
    )
  }
}
```

And voila, just like that, you are rendering on the server both Material UI and Styled Components.
