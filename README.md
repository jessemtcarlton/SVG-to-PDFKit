# SVG-to-PDFKit

## Notes: Added by forker (jacobbubu)

* Using peerDependencies for pdfkit
* adding callback for element drawing.
* support color from color books
  - must use my forked [pdfkit](https://github.com/jacobbubu/pdfkit)
* support customized getComputedStyle
  - Add getComputedStyleCallback

### Example for parseColorCallback

``` js
import SVGtoPDF from "svg-to-pdfkit";
import { load } from "color-books";

const getColor = (bookName, colorName: string) {
  const book = load(bookName);
  if (book.records[colorName]) {
    return {
      name: colorName,
      colorSpace: book.colorSpace,
      components: book.records[colorName].components,
      isSpot: book.isSpot
    };
  }
};

// <path d="..." fill="COLOR_BOOK(PANTONE+ CMYK Coated/PANTONE P 24-8 C)"></path>
const parseColorCallback = (color) => {
  const temp = color.match(/^COLOR_BOOK\((.+)\/(.+)\)$/);
  if (temp) {
    const [bookName, colorName] = temp.slice(1);
    return getColor(bookName, colorName);
  }
};

const draw = (doc, svg, { left, top, width, height }) => {
  doc.save();
  try {
    SVGtoPDF(doc, svg, left, top, {
      width,
      height,
      parseColorCallback
    });
  } finally {
    doc.restore();
  }
};
```

### Example for getComputedStyleCallback

``` coffee
fs = require 'fs'
PDFDocument = require 'pdfkit'
SVGtoPDF = require 'svg-to-pdfkit'
jsdom = require 'jsdom'
{ JSDOM } = jsdom
reduce = Array.prototype.reduce

SvgContent = fs.readFileSync 'svgs/border-trim-css.svg', 'utf8'

styledDOMFromSVGdom = (svg) ->
    dom = new JSDOM svg
    nodeList = dom.window.document.querySelectorAll('style')
    styles = reduce.call nodeList, (sum, curr) ->
        sum += curr.textContent + '\n'
    , ''
    styles = "<style>#{styles}</<style>>"

domForCSSMatch = new JSDOM styledDOMFromSVGdom(SvgContent)

doc = new PDFDocument {
    autoFirstPage: false,
    version: 1.6
}

doc.pipe fs.createWriteStream 'output.pdf'
doc.info.Author = "SkinAT Co., Ltd."

doc.addPage { size: [512, 512], margin: 0 }

getComputedStyleCallback = (node) ->
    className = node.getAttribute('class')
    if className
        tempEle = domForCSSMatch.window.document.createElement 'div'
        tempEle.className = className
        styles = domForCSSMatch.window.getComputedStyle tempEle
        return styles
    else
        {}

parseNodeCallback = (node) ->

SVGtoPDF doc, SvgContent, 0, 0, { width: 200, height: 200, getComputedStyleCallback }

doc.end();
```

Insert SVG into a PDF document created with PDFKit.

#### Use:

    SVGtoPDF(doc, svg, x, y, options);

&nbsp; &nbsp; If you prefer, you can add the function to the PDFDocument prototype:

    PDFDocument.prototype.addSVG = function(svg, x, y, options) {
      return SVGtoPDF(this, svg, x, y, options), this;
    };

&nbsp; &nbsp; And then simply call:

    doc.addSVG(svg, x, y, options);

#### Parameters:

    doc [PDFDocument] = the PDF document created with PDFKit
    svg [SVGElement or string] = the SVG object or XML code
    x, y [number] = the position where the SVG will be added
    options [Object] = >
      - width, height [number] = initial viewport, by default it's the page dimensions
      - preserveAspectRatio [string] = override alignment of the SVG content inside its viewport
      - useCSS [boolean] = use the CSS styles computed by the browser (for SVGElement only)
      - fontCallback [function] = function called to get the fonts, see source code
      - imageCallback [function] = same as above for the images (for Node.js)
      - warningCallback [function] = function called when there is a warning
      - assumePt [boolean] = assume that units are PDF points instead of SVG pixels
      - precision [number] = precision factor for approximative calculations (default = 3)

#### Demos:
&nbsp; &nbsp; <a href="https://alafr.github.io/SVG-to-PDFKit/examples/demo.htm" target="_blank">https://alafr.github.io/SVG-to-PDFKit/examples/demo.htm</a>

&nbsp; &nbsp; <a href="https://alafr.github.io/SVG-to-PDFKit/examples/options.htm" target="_blank">https://alafr.github.io/SVG-to-PDFKit/examples/options.htm</a>

#### Supported:
 - shapes: rect, circle, path, ellipse, line, polyline, polygon
 - special elements: use, nested svg
 - text elements: text, tspan, textPath
 - text attributes: x, y, dx, dy, rotate, text-anchor, textLength, word-spacing, letter-spacing, font-size
 - styling: with attributes only
 - colors: fill, stroke & color (rgb, rgba, hex, string), fill-opacity, stroke-opacity & opacity
 - units: all standard units
 - transformations: transform, viewBox & preserveAspectRatio attributes
 - clip paths & masks
 - images
 - fonts
 - gradients
 - patterns

#### Unsupported:
 - links (<a href="https://github.com/alafr/SVG-to-PDFKit/issues/18">#18</a>)
 - filters
 - text attributes: text-decoration, font-variant, writing-mode, unicode-bidi
 - foreignObject (<a href="https://github.com/alafr/SVG-to-PDFKit/issues/37">#37</a>)
 - other things I don't even know they exist

#### Warning:
 - Use an updated PDFKit version (≥0.8.1): see <a href="https://github.com/alafr/pdfkit/wiki/How-to-install-and-build-a-PDFKit-branch">here</a> how to build it, or use the prebuilt file in the <a href="https://github.com/alafr/SVG-to-PDFKit/tree/master/examples">examples</a> folder.
 - There are bugs, please send issues and/or pull requests.

#### License:
&nbsp; &nbsp; <a href="http://choosealicense.com/licenses/mit/">MIT</a>

#### Other useful projects:
 - <a href="https://github.com/devongovett/pdfkit">PDFKit</a>, the JavaScript PDF generation library for Node and the browser.
 - For inserting SVG graphics into a PDFKit document there is also <a href="https://github.com/devongovett/svgkit">svgkit</a>.
 - For the opposite conversion, from PDF to SVG, you can use <a href="https://github.com/mozilla/pdf.js">Mozilla's PDF.js</a>.
