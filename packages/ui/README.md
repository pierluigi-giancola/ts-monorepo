## Note before start reading

This notes assume this project is used inside as a yarn workspace of a monorepo, you can replicate this same project byt replace all _yarn workspace <...>_ with just _yarn <...>_

Also disable yarn pnp as it cause some problems when running storybook

in .yarnrc.yml
```yml
nodeLinker: "node-modules"
```

## From Scratch

Create package.json file with just the name inside
```json
{
    "name": "blink-ui",
}
```

then
```sh
yarn workspace blink-ui add -P react react-dom
yarn workspace blink-ui -D typescript
```

Some basic folder structure
```
mkdir src
cd src
echo "export * from './components';" > index.ts
mkdir components
touch components/index.ts
```

### Hygen setup (Optional)

Hygen is a tool to automate writing code, this helps mantain the same structure for every component.

To setup the structure of a Component we need to run the init command and then create a template.
```sh
yarn workspace blink-ui -D hygene
yarn workspace blink-ui hygen init self
```

Configuration copied from [here](https://github.com/sixfootsixdesigns/React-Library-Boilerplate/tree/master/_templates/component)

[Documentation about Hygen Templates syntax](https://www.hygen.io/docs/templates)


Now if you run
```sh
yarn workspace blink-ui hygen component with-prompt
```

You will be prompted about the name of the component.
After you input the name, it creates the full component structure inside __src/components__


### Storybook

Storybook allows to render the components in a sandbox and it's super important during development.

```sh
yarn workspace blink-ui add -D @storybook/react
```

Create folder __.storybook__
```sh
mkdir storybook
```

and copy [this file](https://github.com/sixfootsixdesigns/React-Library-Boilerplate/blob/master/.storybook/main.js)


To add sass support install
```sh
yarn workspace blink-ui -D sass-loader@10.1.1 node-sass@5.0.0 style-loader css-loader
```
pinned version because latest (>11 and >6 do not work)

Run Storybook like this
```sh
start-storybook -p 9001 -c .storybook
```

You should be able to see storybook loading correctly

### Typescript

At this point you already run the command to install Typescript in your project.

```sh
yarn workspace blink-ui -D typescript
```

_tsconfig.json_
```json
{
    "compilerOptions": {
        "allowSyntheticDefaultImports": true,
        "declaration": true,
        "esModuleInterop": true,
        "experimentalDecorators": true,
        "jsx": "react",
        "lib": [
            "dom",
            "es5"
        ],
        "module": "esNext",
        "moduleResolution": "node",
        "noImplicitAny": false,
        "noImplicitReturns": true,
        "noUnusedLocals": true,
        "noUnusedParameters": false,
        "outDir": "./dist",
        "pretty": true,
        "sourceMap": true,
        "strict": true,
        "target": "es5"
    },
    "exclude": [
        "node_modules"
    ],
    "include": [
        "./src"
    ]
}
```

**TODO: Study better what every options do and write it down**


### Test: Jest and React Testing Library

Install Jest and RTL:
```sh
yarn workspace blink-ui add -D jest ts-jest @testing-library/react
```

_jest.config.js_
```js
module.exports = {
  preset: 'ts-jest/presets/js-with-ts',
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/?(*.)(spec|test).{js,jsx,ts,tsx}',
  ],
  coveragePathIgnorePatterns: [
    '/node_modules/',
    'dist/',
    '<rootDir>/src/index.ts',
    '<rootDir>/src/components/index.ts',
  ],
  collectCoverageFrom: ['<rootDir>/src/**/*.{js,ts,tsx,jsx}', '!<rootDir>/src/**/*.stories.*'],
  moduleNameMapper: {
    '\\.s?css$': 'identity-obj-proxy',
  },
  coverageThreshold: {
    global: {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
  },
};
```

Running the following command:
```sh
yarn workspace blink-ui jest
```
should result in an error caused by missing type definition of Jest.
```sh
yarn workspace blink-ui add -D @types/jest
```

Now inside our _jest.config.json_ we used a library that we don't install yet.
```sh
yarn workspace blink-ui add -D identity-obj-proxy
```

**TODO: Study better what every options do and write it down**


### Rollup

Install all the libs required.

```sh
yarn workspace blink-ui add -D rollup-plugin-typescript2 rollup-plugin-terser rollup-plugin-postcss @rollup/plugin-commonjs @rollup/plugin-node-resolve rollup-plugin-copy
```

_rollup.config.js_
```js
import typescript from 'rollup-plugin-typescript2';
import { terser } from 'rollup-plugin-terser';
import postcss from 'rollup-plugin-postcss';
import commonjs from '@rollup/plugin-commonjs';
import pkg from './package.json';
import resolve from '@rollup/plugin-node-resolve';
import copy from 'rollup-plugin-copy';
import ts from 'typescript';

export default {
  input: './src/index.ts',
  external: [...Object.keys(pkg.dependencies || {}), ...Object.keys(pkg.peerDependencies || {})],
  output: [
    {
      file: `./dist/${pkg.module}`,
      format: 'es',
      sourcemap: true,
    },
    {
      file: `./dist/${pkg.main}`,
      format: 'cjs',
      sourcemap: true,
    },
  ],
  plugins: [
    resolve(),
    commonjs(),
    postcss(),
    typescript({
      typescript: ts,
      tsconfig: 'tsconfig.json',
      tsconfigDefaults: {
        exclude: [
          '**/*.spec.ts',
          '**/*.test.ts',
          '**/*.stories.ts',
          '**/*.spec.tsx',
          '**/*.test.tsx',
          '**/*.stories.tsx',
          'node_modules',
          'bower_components',
          'jspm_packages',
          'dist',
        ],
        compilerOptions: {
          sourceMap: true,
          declaration: true,
        },
      },
    }),
    terser({
      output: {
        comments: false,
      },
    }),
    copy({
      targets: [
        { src: 'LICENSE', dest: 'dist' },
        { src: 'README.md', dest: 'dist' },
        {
          src: 'package.json',
          dest: 'dist',
          transform: (content) => {
            const { scripts, devDependencies, husky, release, engines, ...keep } = JSON.parse(
              content.toString()
            );
            return JSON.stringify(keep, null, 2);
          },
        },
      ],
    }),
  ],
};

```