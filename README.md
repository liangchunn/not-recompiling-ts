# not-recompiling-ts

Webpack does not recompile when ambient type definition files are changed. This leads to a 'phantom type' being persisted across incremental builds, leading to no errors in the CLI, but errors being reported on VSCode and during a cold build.

## Steps to reproduce
1. Run `yarn start`
2. Remove `b: string` in `src/types.ts` and save
3. Watch CLI, no errors.
4. <kbd>CTRL+C</kbd> and `yarn start` -> error

## Dependencies
- `webpack@4.12.0`
- `ts-loader@4.4.1`
- `typescript@2.9.2`

## Webpack config file
The webpack configuration lives inside the `typescript-node-scripts` package: `node_modules/typescript-node-scripts/lib/webpack.config.dev.js`

```ts
const paths = require('./paths')
const nodeExternals = require('webpack-node-externals')
const tsconfigPathsPlugin = require('tsconfig-paths-webpack-plugin')
const formatTsLoaderMessages = require('./formatTsLoaderMessages')
const { tslintShouldEmitErrors } = require('./tsLintHelper')

module.exports = {
    mode: 'development',
    entry: paths.appIndexJs,
    target: 'node',
    externals: [nodeExternals()],
    devtool: 'inline-source-map',
    output: {
        path: paths.appDevBundlePath,
        filename: 'bundle.js',
        libraryTarget: 'commonjs',
    },
    resolve: {
        extensions: ['.ts', '.tsx', '.js', 'jsx', '.json'],
        plugins: [
            new tsconfigPathsPlugin({
                configFile: paths.appTsConfig,
            }),
        ],
    },
    module: {
        rules: [
            {
                test: /\.(t|j)sx?$/,
                enforce: 'pre',
                use: [
                    {
                        loader: require.resolve('tslint-loader'),
                        options: {
                            emitErrors: tslintShouldEmitErrors(paths.appTsLint),
                            tsConfigFile: paths.appTsConfig,
                            configFile: paths.appTsLint,
                            formatter: 'lintTable',
                            formattersDirectory: __dirname + '/formatters/',
                        },
                    },
                ],
            },
            {
                test: /\.(t|j)sx?$/,
                include: paths.appSrc,
                loader: require.resolve('ts-loader'),
                options: {
                    colors: true,
                    errorFormatter: formatTsLoaderMessages,
                },
            },
        ],
    },
    optimization: {
        nodeEnv: false,
    },
    node: {
        __dirname: false,
        __filename: false,
    },
    performance: {
        hints: false,
    },
}
```