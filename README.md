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
		\node(d) { $d$ };
		\node(1) [right of = d] { $1$ };
		\draw[->] (d) -- (1);
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
			<div style="width: ${w} height: ${h}">
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

```
<p> Here are two nodes connected with each other </p>
<div style="width: 0pt height: 0pt">
<svg width=0pt height=0pt viewBox="-72 -72 0 0"><g transform="translate(-72.27000427246092,-72.27000427246092) scale(1,-1)"><g> <g stroke="rgb(0.0%,0.0%,0.0%)"> <g fill="rgb(0.0%,0.0%,0.0%)"> <g stroke-width="0.4pt"> <g> <text alignment-baseline="baseline" y="-72.27000427246092" x="-72.27000427246092" font-family="cmmi10" font-size="10" fill="black">d</text><text alignment-baseline="baseline" y="-72.27000427246092" x="-67.06512641906737" font-family="cmr10" font-size="10" fill="black">1</text></g> </g> </g> </g> </g> </g></svg>
</div>
```
