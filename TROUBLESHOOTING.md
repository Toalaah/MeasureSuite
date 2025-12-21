## Building for TS

- When running `npm i`, node-gyp errors out with
```
../src/binding/measuresuite_binding.c:18:10: fatal error: node/node_api.h: No such file or directory
   18 | #include <node/node_api.h>
      |          ^~~~~~~~~~~~~~~~~
compilation terminated.
```
- This means that the `node/node_api.h`-header file could not be found.
- Try using `find / -type f -name 'node_api.h'` to find the file and  
    - either add the path to the `node` folder to the `include_dirs` in `binding.gyp` in the `measuresuite` target;
    - or use `CFLAGS=-I/path/to/node npm i` while installing.
- If `node/node_api.h` could not be found, get the node-package from [the node.js website](https://nodejs.org/en/download/current/) extract, find the `node_api.h` in there and set the paths accordingly.


## Erroneous Cycle Count Measurements

When running `make check` or using MeasureSuite in general, it is possible that you may encounter spurious errors when collecting cycle counts. If you get errors when running running `./lib/test/integration/itest_json_asm.tst` and/or the test reports cycle counts of 0:

```
{
  "stats": {
    "numFunctions": 4,
    "runtime": 0,
    "incorrect": 0,
    "timer": "PMC"
  },
  "functions": [
    {
      "type": "ASM",
      "chunks": 0
    },
    {
      "type": "BIN"
    },
    {
      "type": "SHARED_OBJECT"
    },
    {
      "type": "ELF"
    }
  ],
  "cycles": [
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
  ]
}
```

This may be due to the measurements running on e-cores. Try manually setting the CPU affinity to "real" cores only and re-run the test, e.g via `taskset -c $REAL_CORE_ID make check`. Your real cores can be found by running `lscpu --all --extended`; if the core has at least 2 CPUs it is likely a p-core, otherwise it is probably an e-core. In a programatic environment, `<sched.h>` provides an API for configuring CPU affinity using `sched_getaffinity`, though this requires defining `_GNU_SOURCE`.


## Running C Tests

- When running `make check`
- Some tests are skipped.
- Reason: Those require the CPU flags `BMI2` and `ADX`.
- Resolution: You can check with `lscpu | grep -e bmi2 -e adx` if they are available. If yes, [open a new Github Issue](https://github.com/0xADE1A1DE/MeasureSuite/issues/new). If not, run the tests on a different CPU.
 

## Running TS Tests

- When running `npm test`
- Error: `vitest: command not found`
- Reason: Node.js dependencies are not installed (i.e. `node_modules` does not exist)
- Resolution: `npm i`

