App Router

...

Styling

CSS-in-JS
CSS-in-JS
Warning: CSS-in-JS libraries which require runtime JavaScript are not currently supported in Server Components. Using CSS-in-JS with newer React features like Server Components and Streaming requires library authors to support the latest version of React, including concurrent rendering

.
We're working with the React team on upstream APIs to handle CSS and JavaScript assets with support for React Server Components and streaming architecture.
The following libraries are supported in Client Components in the app directory (alphabetical):
* 		kuma-ui
* 		@mui/material
* 		pandacss
* 		styled-jsx
* 		styled-components
* 		style9
* 		tamagui
* 		tss-react
* 		vanilla-extract
The following are currently working on support:
* 		emotion
Good to know: We're testing out different CSS-in-JS libraries and we'll be adding more examples for libraries that support React 18 features and/or the app directory.
If you want to style Server Components, we recommend using CSS Modules or other solutions that output CSS files, like PostCSS or Tailwind CSS.
Configuring CSS-in-JS in app

Configuring CSS-in-JS is a three-step opt-in process that involves:
* 		A style registry to collect all CSS rules in a render.
* 		The new useServerInsertedHTML hook to inject rules before any content that might use them.
* 		A Client Component that wraps your app with the style registry during initial server-side rendering.
styled-jsx

Using styled-jsx in Client Components requires using v5.1.0. First, create a new registry:

app/registry.js
JavaScript



'use client'
 
import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { StyleRegistry, createStyleRegistry } from 'styled-jsx'
 
export default function StyledJsxRegistry({ children }) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [jsxStyleRegistry] = useState(() => createStyleRegistry())
 
  useServerInsertedHTML(() => {
    const styles = jsxStyleRegistry.styles()
    jsxStyleRegistry.flush()
    return <>{styles}</>
  })
 
  return <StyleRegistry registry={jsxStyleRegistry}>{children}</StyleRegistry>
}
Then, wrap your root layout with the registry:

app/layout.js
JavaScript



import StyledJsxRegistry from './registry'
 
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <StyledJsxRegistry>{children}</StyledJsxRegistry>
      </body>
    </html>
  )
}
View an example here

.
Styled Components

Below is an example of how to configure styled-components@6 or newer:
First, use the styled-components API to create a global registry component to collect all CSS style rules generated during a render, and a function to return those rules. Then use the useServerInsertedHTML hook to inject the styles collected in the registry into the <head> HTML tag in the root layout.

lib/registry.js
JavaScript



'use client'
 
import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'
 
export default function StyledComponentsRegistry({ children }) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet())
 
  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement()
    styledComponentsStyleSheet.instance.clearTag()
    return <>{styles}</>
  })
 
  if (typeof window !== 'undefined') return <>{children}</>
 
  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  )
}
Wrap the children of the root layout with the style registry component:

app/layout.js
JavaScript



import StyledComponentsRegistry from './lib/registry'
 
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <StyledComponentsRegistry>{children}</StyledComponentsRegistry>
      </body>
    </html>
  )
}
View an example here

.
Good to know:
* 		During server rendering, styles will be extracted to a global registry and flushed to the <head> of your HTML. This ensures the style rules are placed before any content that might use them. In the future, we may use an upcoming React feature to determine where to inject the styles.
* 		During streaming, styles from each chunk will be collected and appended to existing styles. After client-side hydration is complete, styled-components will take over as usual and inject any further dynamic styles.
* 		We specifically use a Client Component at the top level of the tree for the style registry because it's more efficient to extract CSS rules this way. It avoids re-generating styles on subsequent server renders, and prevents them from being sent in the Server Component payload.
Previous
Tailwind CSS

Next
Sass

Sass	
	

Next.js has built-in support for Sass using both the .scss and .sass extensions. You can use component-level Sass via CSS Modules and the .module.scssor .module.sass extension.
First, install sass

:

Terminal


npm install --save-dev sass
Good to know:
Sass supports two different syntax

, each with their own extension. The .scss extension requires you use the SCSS syntax

, while the .sass extension requires you use the Indented Syntax ("Sass")

.
If you're not sure which to choose, start with the .scss extension which is a superset of CSS, and doesn't require you learn the Indented Syntax ("Sass").
Customizing Sass Options

If you want to configure the Sass compiler, use sassOptions in next.config.js.

next.config.js


const path = require('path')
 
module.exports = {
  sassOptions: {
    includePaths: [path.join(__dirname, 'styles')],
  },
}
Sass Variables

Next.js supports Sass variables exported from CSS Module files.
For example, using the exported primaryColor Sass variable:

app/variables.module.scss


$primary-color: #64ff00;
 
:export {
  primaryColor: $primary-color;
}

app/page.js


// maps to root `/` URL
 
import variables from './variables.module.scss'
 
export default function Page() {
  return <h1 style={{ color: variables.primaryColor }}>Hello, Next.js!</h1>
}
Previous
CSS-in-JS

Next
Optimizing
Optimizing 	
	

App Router

Building Your Application

Optimizing
Optimizations
Next.js comes with a variety of built-in optimizations designed to improve your application's speed and Core Web Vitals

. This guide will cover the optimizations you can leverage to enhance your user experience.
Built-in Components

Built-in components abstract away the complexity of implementing common UI optimizations. These components are:
* 		Images: Built on the native <img> element. The Image Component optimizes images for performance by lazy loading and automatically resizing images based on device size.
* 		Link: Built on the native <a> tags. The Link Component prefetches pages in the background, for faster and smoother page transitions.
* 		Scripts: Built on the native <script> tags. The Script Component gives you control over loading and execution of third-party scripts.
Metadata

Metadata helps search engines understand your content better (which can result in better SEO), and allows you to customize how your content is presented on social media, helping you create a more engaging and consistent user experience across various platforms.
The Metadata API in Next.js allows you to modify the <head> element of a page. You can configure metadata in two ways:
* 		Config-based Metadata: Export a static metadata object or a dynamic generateMetadata function in a layout.js or page.js file.
* 		File-based Metadata: Add static or dynamically generated special files to route segments.
Additionally, you can create dynamic Open Graph Images using JSX and CSS with imageResponse constructor.
Static Assets

Next.js /public folder can be used to serve static assets like images, fonts, and other files. Files inside /public can also be cached by CDN providers so that they are delivered efficiently.
Analytics and Monitoring

For large applications, Next.js integrates with popular analytics and monitoring tools to help you understand how your application is performing. Learn more in the OpenTelemetry and Instrumentation guides.
Images
Optimize your images with the built-in `next/image` component.
Fonts
Optimize your application's web fonts with the built-in `next/font` loaders.
Scripts
Optimize 3rd party scripts with the built-in Script component.
Metadata
Use the Metadata API to define metadata in any layout or page.
Static Assets
Next.js allows you to serve static files, like images, in the public directory. You can learn how it works here.
Lazy Loading
Lazy load imported libraries and React Components to improve your application's loading performance.
Analytics
Measure and track page performance using Next.js Speed Insights
OpenTelemetry
Learn how to instrument your Next.js app with OpenTelemetry.
Instrumentation
Learn how to use instrumentation to run code at server startup in your Next.js app
Previous
Sass

Next
Images
Images	
	

Image Optimization
Examples
* 		Image Component
According to Web Almanac

, images account for a huge portion of the typical website’s page weight

and can have a sizable impact on your website's LCP performance

.
The Next.js Image component extends the HTML <img> element with features for automatic image optimization:
* 		Size Optimization: Automatically serve correctly sized images for each device, using modern image formats like WebP and AVIF.
* 		Visual Stability: Prevent layout shift automatically when images are loading.
* 		Faster Page Loads: Images are only loaded when they enter the viewport using native browser lazy loading, with optional blur-up placeholders.
* 		Asset Flexibility: On-demand image resizing, even for images stored on remote servers
🎥 Watch: Learn more about how to use next/image → YouTube (9 minutes)

.
Usage



import Image from 'next/image'
You can then define the src for your image (either local or remote).
Local Images

To use a local image, import your .jpg, .png, or .webp image files.
Next.js will automatically determine the width and height of your image based on the imported file. These values are used to prevent Cumulative Layout Shift

while your image is loading.

app/page.js


import Image from 'next/image'
import profilePic from './me.png'
 
export default function Page() {
  return (
    <Image
      src={profilePic}
      alt="Picture of the author"
      // width={500} automatically provided
      // height={500} automatically provided
      // blurDataURL="data:..." automatically provided
      // placeholder="blur" // Optional blur-up while loading
    />
  )
}
Warning: Dynamic await import() or require() are not supported. The import must be static so it can be analyzed at build time.
Remote Images

To use a remote image, the src property should be a URL string.
Since Next.js does not have access to remote files during the build process, you'll need to provide the width, height and optional blurDataURL props manually.
The width and height attributes are used to infer the correct aspect ratio of image and avoid layout shift from the image loading in. The width and height do not determine the rendered size of the image file. Learn more about Image Sizing.

app/page.js


import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
To safely allow optimizing images, define a list of supported URL patterns in next.config.js. Be as specific as possible to prevent malicious usage. For example, the following configuration will only allow images from a specific AWS S3 bucket:

next.config.js


module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
      },
    ],
  },
}
Learn more about remotePatterns configuration. If you want to use relative URLs for the image src, use a loader.
Domains

Sometimes you may want to optimize a remote image, but still use the built-in Next.js Image Optimization API. To do this, leave the loader at its default setting and enter an absolute URL for the Image src prop.
To protect your application from malicious users, you must define a list of remote hostnames you intend to use with the next/image component.
Learn more about remotePatterns configuration.
Loaders

Note that in the example earlier, a partial URL ("/me.png") is provided for a local image. This is possible because of the loader architecture.
A loader is a function that generates the URLs for your image. It modifies the provided src, and generates multiple URLs to request the image at different sizes. These multiple URLs are used in the automatic srcset

generation, so that visitors to your site will be served an image that is the right size for their viewport.
The default loader for Next.js applications uses the built-in Image Optimization API, which optimizes images from anywhere on the web, and then serves them directly from the Next.js web server. If you would like to serve your images directly from a CDN or image server, you can write your own loader function with a few lines of JavaScript.
You can define a loader per-image with the loader prop, or at the application level with the loaderFile configuration.
Priority

You should add the priority property to the image that will be the Largest Contentful Paint (LCP) element

for each page. Doing so allows Next.js to specially prioritize the image for loading (e.g. through preload tags or priority hints), leading to a meaningful boost in LCP.
The LCP element is typically the largest image or text block visible within the viewport of the page. When you run next dev, you'll see a console warning if the LCP element is an <Image> without the priority property.
Once you've identified the LCP image, you can add the property like this:

app/page.js


import Image from 'next/image'
import profilePic from '../public/me.png'
 
export default function Page() {
  return <Image src={profilePic} alt="Picture of the author" priority />
}
See more about priority in the next/image component documentation.
Image Sizing

One of the ways that images most commonly hurt performance is through layout shift, where the image pushes other elements around on the page as it loads in. This performance problem is so annoying to users that it has its own Core Web Vital, called Cumulative Layout Shift

. The way to avoid image-based layout shifts is to always size your images

. This allows the browser to reserve precisely enough space for the image before it loads.
Because next/image is designed to guarantee good performance results, it cannot be used in a way that will contribute to layout shift, and must be sized in one of three ways:
* 		Automatically, using a static import
* 		Explicitly, by including a width and height property
* 		Implicitly, by using fill which causes the image to expand to fill its parent element.
What if I don't know the size of my images?
If you are accessing images from a source without knowledge of the images' sizes, there are several things you can do:
Use fill
The fill prop allows your image to be sized by its parent element. Consider using CSS to give the image's parent element space on the page along sizes prop to match any media query break points. You can also use object-fit

with fill, contain, or cover, and object-position

to define how the image should occupy that space.
Normalize your images
If you're serving images from a source that you control, consider modifying your image pipeline to normalize the images to a specific size.
Modify your API calls
If your application is retrieving image URLs using an API call (such as to a CMS), you may be able to modify the API call to return the image dimensions along with the URL.
If none of the suggested methods works for sizing your images, the next/image component is designed to work well on a page alongside standard <img> elements.
Styling

Styling the Image component is similar to styling a normal <img> element, but there are a few guidelines to keep in mind:
* 		Use className or style, not styled-jsx.
    * 		In most cases, we recommend using the className prop. This can be an imported CSS Module, a global stylesheet, etc.
    * 		You can also use the style prop to assign inline styles.
    * 		You cannot use styled-jsx because it's scoped to the current component (unless you mark the style as global).
* 		When using fill, the parent element must have position: relative
    * 		This is necessary for the proper rendering of the image element in that layout mode.
* 		When using fill, the parent element must have display: block
    * 		This is the default for <div> elements but should be specified otherwise.
Examples

Responsive






import Image from 'next/image'
import mountains from '../public/mountains.jpg'
 
export default function Responsive() {
  return (
    <div style={{ display: 'flex', flexDirection: 'column' }}>
      <Image
        alt="Mountains"
        // Importing an image will
        // automatically set the width and height
        src={mountains}
        sizes="100vw"
        // Make the image display full width
        style={{
          width: '100%',
          height: 'auto',
        }}
      />
    </div>
  )
}
Fill Container






import Image from 'next/image'
import mountains from '../public/mountains.jpg'
 
export default function Fill() {
  return (
    <div
      style={{
        display: 'grid',
        gridGap: '8px',
        gridTemplateColumns: 'repeat(auto-fit, minmax(400px, auto))',
      }}
    >
      <div style={{ position: 'relative', height: '400px' }}>
        <Image
          alt="Mountains"
          src={mountains}
          fill
          sizes="(min-width: 808px) 50vw, 100vw"
          style={{
            objectFit: 'cover', // cover, contain, none
          }}
        />
      </div>
      {/* And more images in the grid... */}
    </div>
  )
}
Background Image






import Image from 'next/image'
import mountains from '../public/mountains.jpg'
 
export default function Background() {
  return (
    <Image
      alt="Mountains"
      src={mountains}
      placeholder="blur"
      quality={100}
      fill
      sizes="100vw"
      style={{
        objectFit: 'cover',
      }}
    />
  )
}
For examples of the Image component used with the various styles, see the Image Component Demo

.
Other Properties

View all properties available to the next/image component.
Configuration

The next/image component and Next.js Image Optimization API can be configured in the next.config.js file. These configurations allow you to enable remote images, define custom image breakpoints, change caching behavior and more.
Read the full image configuration documentation for more information.
API Reference
Learn more about the next/image API.
App Router

...

Components
<Image>
Optimize Images in your Next.js Application using the built-in `next/image` Component.
Previous
Optimizing

Next
Fonts
