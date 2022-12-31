---
title: "Setting up TailwindCSS in a Parcel project"
---

#### Initialize the project

```
mkdir tailwindcss-parcel-setup-example
tailwindcss-parcel-setup-example
yarn add -D parcel
mkdir src
touch src/index.html
```

#### Install TailwindCSS
```
yarn add -D tailwindcss postcss
npx tailwindcss init
```


#### Configure PostCSS
```json
{
	"plugins": {
		"tailwindcss": {}
	}
}
```

#### Configure template paths

```js
module.exports = {
	content: [
		"./src/**/*.{html,js,ts,jsx,tsx}",
	],
	theme: {
		extend: {}
	},
	plugins: [],
}
```

Read more about `content` option [here](https://tailwindcss.com/docs/content-configuration).

#### Add the tailwindcss directives to your CSS
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### Start your build process

```
npx parcel src/index.html
```

#### Start using Tailwind in your project

```html
<!doctype html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link href="./index.css" rel="stylesheet">
</head>
<body>
  <h1 class="text-3xl font-bold underline">
    Hello world!
  </h1>
</body>
</html>
```