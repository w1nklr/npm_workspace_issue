# npm script error handling

This project tests the conditions for which error handling is broken for `npm` scripts.

The issue:

```
"scripts": {
    "test": "npm run fail && echo ERROR: echo is wrongly executed after previous failure",
    "fail": "1 && echo Correctly not executed after failure"
  },
```

Running `npm run fail` is expected to return an error code, thus preventing `npm run test` or `npm run fail && echo ERROR` to continue execution after the error.

The issue is that `npm run` does not return the error status of the script.

This issue only occurs in workspaces, starting with node version `20.0.0`.
Node versions `18.20.8` and `19.9.0` (last releases of their branch as of my writing) do not have this behavior.

This prevents tools, like `nx`, to report errors while the scripts in the workspaces are failing.

## Testing various node/npm versions

`volta` is used to specify the node version to use.

It seems that specifying `npm` version does not affect the experienced behavior.

## Tests

### Test from workspace root

`npm run test --workspaces` or `npm run --prefix workspaces/ws_1 test`.

Only the node version specified in the workspace root is taken into account. The node version in the workspace itself (directory `workspaces/ws_1`) does not have an impact.

Note that `npm run --prefix non_workspaces/lib_1 test` behaves correctly.

### Test from single workspace

`npm run test`.

Only the node version specified in the workspace (directory `workspaces/ws_1`) is taken into account. The node version in the workspace root does not have an impact.

Note that `npm run test` behaves correctly in other subdirectories that are not workspaces.

## Output

Expected output:

```
$ npm run test

> workspace_root@1.0.0 test
> npm run fail && echo ERROR: echo is wrongly executed after previous failure


> workspace_root@1.0.0 fail
> 1 && echo Correctly not executed after failure

'1' is not recognized as an internal or external command,
operable program or batch file.
```

Output:

```
$ npm run test

> workspace_root@1.0.0 test
> npm run fail && echo ERROR: echo is wrongly executed after previous failure


> workspace_root@1.0.0 fail
> 1 && echo Correctly not executed after failure

'1' is not recognized as an internal or external command,
operable program or batch file.
ERROR: echo is wrongly executed after previous failure
```

## node and npm versions

Last releases extracted from https://nodejs.org/en/about/previous-releases, as of 2025/05/16.

| Node.js branch | Node version | npm version |
| -------------- | ------------ | ----------- |
| v24            | v24.0.2      | v11.3.0     |
| v22            | v22.15.1     | v10.9.2     |
| v20            | v20.19.2     | v10.8.2     |
| v19            | v19.9.0      | v9.6.3      |
| v18            | v18.20.8     | v10.8.2     |
