# Logging

wolfEngine supports output of log messages for informative and debug purposes. To enable debug logging, wolfEngine must first be compiled with debug support enabled. If using Autoconf, this is done using the `--enable-debug` option to `./configure`:
```
./configure --enable-debug
```

If not using Autoconf/configure, define `WOLFENGINE_DEBUG` when compiling the wolfEngine library.

## Enable / Disable Debug Logging

Once debug support has been compiled into the library, debugging must be enabled at runtime using the wolfEngine control commands specified in Section 5. To enable debug logging, use the `enable_debug` control command with the value of "1" to enable logging or "0" to disable logging. To enable logging using the `ENGINE_ctrl_cmd()` API:
```
int ret = 0;
ret = ENGINE_ctrl_cmd(e, “enable_debug”, 1, NULL, NULL, 0);
if (ret != 1) {
    printf(“Failed to enable debug logging\n”);
}
```

If wolfEngine has not been compiled with debug support enabled, an attempt to set `enable_debug` with `ENGINE_ctrl_cmd()` will return failure (0).

## Controlling Logging Levels

wolfEngine supports the following logging levels. These are defined in the “include/wolfengine/we_logging.h” header file as part of the wolfEngine_LogType enum:

| Log Enum | Description | Log Enum Value | 
| -------------- |  --------------- |--------------------- |
| WE_LOG_ERROR | Logs errors | 0x0001 |
| WE_LOG_ENTER | Logs when entering functions | 0x0002 |
| WE_LOG_LEAVE | Logs when leaving functions | 0x0004 |
| WE_LOG_INFO | Logs informative messages | 0x0008 |
| WE_LOG_VERBOSE | Verbose logs, including encrypted/decrypted/digested data | 0x0010 |
| WE_LOG_LEVEL_DEFAULT | Default log level, all except verbose level | WE_LOG_ERROR &#124; WE_LOG_ENTER &#124; WE_LOG_LEAVE &#124; WE_LOG_INFO |
WE_LOG_LEVEL_ALL WE_LOG_ERROR | All log levels are enabled | WE_LOG_ENTER &#124; WE_LOG_LEAVE &#124; WE_LOG_INFO &#124; WE_LOG_VERBOSE |


The default wolfEngine logging level includes `WE_LOG_ERROR`, `WE_LOG_ENTER`, `WE_LOG_LEAVE`, and `WE_LOG_INFO`. This includes all log levels except verbose logs (`WE_LOG_VERBOSE`).

Log levels can be controlled using the "**log_level**" engine control command at runtime, either through the `ENGINE_ctrl_cmd()` API or OpenSSL configuration file settings. For example, to turn on only error and informative logs using the “log_level” control command, an application would call:
```
#include <wolfengine/we_logging.h>

ret = ENGINE_ctrl_cmd(e, “log_level”, WE_LOG_ERROR | WE_LOG_INFO,
NULL, NULL, 0);
if (ret != 1) {
    printf(“Failed to set logging level\n”);
}
```

## Controlling Component Logging

wolfEngine allows logging on a per-component basis. Components are defined in the wolfEngine_LogComponents enum in `include/wolfengine/we_logging.h`:

| Log Component Enum | Description | Component Enum Value |
| ------------------------------ | --------------- | -------------------------------- |
| WE_LOG_RNG | Random number generation | 0x0001 |
| WE_LOG_DIGEST | Digests (SHA-1/2/3) | 0x0002 |
| WE_LOG_MAC | MAC functions (HMAC, CMAC) | 0x0004 |
| WE_LOG_CIPHER | Ciphers (AES, 3DES) | 0x0008 |
| WE_LOG_PK | Public Key Algorithms (RSA, ECC) | 0x0010 |
| WE_LOG_KE | Key Agreement Algorithms (DH, ECDH) | 0x0020 |
| WE_LOG_ENGINE | All engine specific logs | 0x0040 |
| WE_LOG_COMPONENTS_ALL | Log all components | WE_LOG_RNG &#124; WE_LOG_DIGEST &#124; WE_LOG_MAC &#124; WE_LOG_CIPHER &#124; WE_LOG_PK &#124; WE_LOG_KE &#124; WE_LOG_ENGINE |
| WE_LOG_COMPONENTS_DEFAULT | Default components logged (all). | WE_LOG_COMPONENTS_ALL |


The default wolfEngine logging configuration logs all components (`WE_LOG_COMPONENTS_DEFAULT`).

Components logged can be controlled using the “ **log_components** ” engine control command at runtime, either through the `ENGINE_ctrl_cmd()` API or OpenSSL configuration file settings. For example, to turn on only logging only for the Digest and Cipher algorithms:
```
#include <wolfengine/we_logging.h>

ret = ENGINE_ctrl_cmd(e, “ **log_components** ”, WE_LOG_DIGEST | WE_LOG_CIPHER,
NULL, NULL, 0);
if (ret != 1) {
    printf(“Failed to set log components\n”);
}
```
## Setting a Custom Logging Callback

By default wolfEngine outputs debug log messages using **fprintf()** to **stderr**.

Applications that want to have more control over how or where log messages are output can write and register a custom logging callback with wolfEngine. The logging callback should match the prototype of wolfEngine_Logging_cb in `include/wolfengine/we_logging.h`:
```
/**
* wolfEngine logging callback.
* logLevel - [IN] - Log level of message
* component - [IN] - Component that log message is coming from
* logMessage - [IN] - Log message
*/
typedef void (* **wolfEngine_Logging_cb** )(const int logLevel,
const int component,
const char *const logMessage);
```
The callback can then be registered with wolfEngine using the “ **set_logging_cb** ” engine control command. For example, to use the `ENGINE_ctrl_cmd()` API to set a custom logging callback:
```
void **customLogCallback** (const int logLevel, const int component,
const char* const logMessage)
{
    (void)logLevel;
    (void)component;
    fprintf(stderr, “wolfEngine log message: %d\n”, logMessage);
}

int **main** (void)
{
    int ret;
    ENGINE* e;
...
    ret = ENGINE_ctrl_cmd(e, “ **set_logging_cb** ”, 0, NULL,
    (void(*)(void))my_Logging_cb, 0);
    if (ret != 1) {
        /* failed to set logging callback */
    }
...
}
```
