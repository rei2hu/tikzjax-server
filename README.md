# tikzjax-server

This is some code based off of [tikzjax](https://github.com/kisonecat/tikzjax)
to generate tikz code to svgs in node.js environments (specifically for html
document generation in my case).

## example

```js
const { load: wasmLoad } = require("tikzjax-server");

let content = `
	<p> Here are two nodes connected with each other </p>
	\\begin{tikzpicture}
		[nodes={draw, circle, minimum size=1cm}, node distance={40mm}]
		\\node(d) { $d$ };
		\\node(1) [right of = d] { $1$ };
		\\draw[->] (d) -- (1);
	\\end{tikzpicture}
`;

wasmLoad.then(async generator => {
	// script will only pre/apppend \begin{document} \end{document} commands
	// so send it content surrounded by \begin{tikzpicture} \end{tikzpicture}
	const matches =
		content.match(/(\\begin{tikzpicture}(\n|.)+?\\end{tikzpicture})/g) ??
		[];

	for (const match of matches) {
		const { machine, html } = await generator(match);

		const w = `${machine.paperwidth}pt`;
		const h = `${machine.paperheight}pt`;

		// this is probably the minimal amount of wrapping you want to do to
		// have the image show up nicely in an html document
		content = content.replace(
			match,
			`
			<div style="width: ${w}; height: ${h}">
				${html.replace(
					"<svg>",
					`<svg width=${w} height=${h} viewBox="-72 -72 ${machine.paperwidth} ${machine.paperheight}">`
				)}
			</div>
		`
		);
	}

	console.log(content);
});
```

## output

```html
 <p> Here are two nodes connected with each other </p>

                        <div style="width: 142.66374pt; height: 28.85272pt">
                                <svg width=142.66374pt height=28.85272pt viewBox="-72 -72 142.66374 28.85272"><g transform="translate(-57.84364318847655,-57.84364318847655) scale(1,-1)"><g> <g stroke="rgb(0.0%,0.0%,0.0%)"> <g fill="rgb(0.0%,0.0%,0.0%)"> <g stroke-width="0.4pt"> <g> <g> <path d=" M  14.22636 0.0 C  14.22636 7.8571 7.8571 14.22636 0.0 14.22636 C  -7.8571 14.22636 -14.22636 7.8571 -14.22636 0.0 C  -14.22636 -7.8571 -7.8571 -14.22636 0.0 -14.22636 C  7.8571 -14.22636 14.22636 -7.8571 14.22636 0.0 Z M  0.0 0.0  " style="fill:none"/> <g> <g transform="matrix(1.0,0.0,0.0,1.0,-2.60243,-3.47221)"> <g stroke="rgb(0.0%,0.0%,0.0%)"> <g fill="rgb(0.0%,0.0%,0.0%)"> <g stroke="none" transform="scale(-1,1) translate(-57.84364318847655,-57.84364318847655) scale(-1,-1)"><text alignment-baseline="baseline" y="-57.84364318847655" x="-57.84364318847655" font-family="cmmi10" font-size="10" fill="black">d</text></g></g> </g> </g> </g> </g> <g> <path d=" M  128.03738 0.0 C  128.03738 7.8571 121.66812 14.22636 113.81102 14.22636 C  105.95392 14.22636 99.58466 7.8571 99.58466 0.0 C  99.58466 -7.8571 105.95392 -14.22636 113.81102 -14.22636 C  121.66812 -14.22636 128.03738 -7.8571 128.03738 0.0 Z M  113.81102 0.0  " style="fill:none"/> <g> <g transform="matrix(1.0,0.0,0.0,1.0,111.31102,-3.22221)"> <g stroke="rgb(0.0%,0.0%,0.0%)"> <g fill="rgb(0.0%,0.0%,0.0%)"> <g stroke="none" transform="scale(-1,1) translate(-57.84364318847655,-57.84364318847655) scale(-1,-1)"><text alignment-baseline="baseline" y="-57.84364318847655" x="-57.84364318847655" font-family="cmr10" font-size="10" fill="black">1</text></g></g> </g> </g> </g> </g> <path d=" M  14.42636 0.0 L  98.92467 0.0  " style="fill:none"/> <g> <g transform="matrix(1.0,0.0,0.0,1.0,98.92467,0.0)"> <g> <g stroke-width="0.31999pt"> <g stroke-dasharray="none" stroke-dashoffset="0.0pt"> <g stroke-linecap="round"> <g stroke-linejoin="round"> <path d=" M  -1.19998 1.59998 C  -1.09998 0.99998 0.0 0.09999 0.29999 0.0 C  0.0 -0.09999 -1.09998 -0.99998 -1.19998 -1.59998  " style="fill:none"/> </g> </g> </g> </g> </g>  </g> </g> </g> </g> </g> </g> </g> </g></svg>
                        </div>
```
