# 使用 MAVSDK 进行集成测试

可以使用基于 [MAVSDK](https://mavsdk.mavlink.io) 的集成测试对 PX4 进行端到端测试。

目前主要针对 SITL 开发测试，并在持续集成（CI）中运行。 但是，它们最终旨在推广到实际测试。

测试需要将MAVSAK C++库安装到系统目录（如： `/usr/lib` or `/usr/local/lib`）

## 安装 MAVSDK C++ 库

### 运行所有PX4测试

二进行安装或源码安装：
- Install the development toolchain for [Linux](../dev_setup/dev_env_linux_ubuntu.md) or [macOS](../dev_setup/dev_env_mac.md) (Windows not supported). Gazebo is required, and should be installed by default.
- [Get the PX4 source code](../dev_setup/building_px4.md#download-the-px4-source-code):

  ```sh
  git clone https://github.com/PX4/PX4-Autopilot.git --recursive
  cd PX4-Autopilot
  ```


### Build PX4 for Testing

使用以下命令构建 PX4 源码：

```sh
DONT_RUN=1 make px4_sitl gazebo mavsdk_tests 
```

### Install the MAVSDK C++ Library

运行 [sitl.json](https://github.com/PX4/PX4-Autopilot/blob/master/test/mavsdk_tests/configs/sitl.json) 中定义的所有SITL测试，执行：

要看所有可用的命令行参数，运行：
- [MAVSDK > Installation > C++](https://mavsdk.mavlink.io/develop/en/getting_started/installation.html#cpp): Install as a prebuilt library on supported platforms (recommended)
- [MAVSDK > Contributing > Building from Source](https://mavsdk.mavlink.io/develop/en/contributing/build.html#build_sdk_cpp): Build  C++ library from source.

## 准备 PX4 源码

使用的术语：

```sh
test/mavsdk_tests/mavsdk_test_runner.py test/mavsdk_tests/configs/sitl.json --speed-factor 10
```

This will list all of the tests and then run them sequentially.


To see all possible command line arguments use the `-h` argument:

```sh
test/mavsdk_tests/mavsdk_test_runner.py -h

用法：mavsdk_test_runner。 y [-h] [--log-dir LOG_DIR] [--speed-factor SPEED_FACTOR] [--trerations ITERATION] [--abort-early] [--gui] [--model MODEL]
                             [--case CASE] [--debugger DEBUGER] [--verbose]
                             config_file

posital 参数：
  config_file JSON 使用的JSON配置文件

optional 参数：
  -h, --help 显示此帮助信息并退出
  --log-dir LOG_DIR 日志文件目录
  --speed-factor SPEED_FACTOR
                        模拟运行的速度因子
  --迭代ITERATION
                        在首次失败的测试中运行所有测试的频率
  --abort-early 中止
  --guide 显示模拟的可视化化
  MODEL 只为一个模型运行测试
  --case CASE 只运行测试一个案例
  --debugger DEBUGER 调试器：callgrind, gdb, lldb
  --verbose 启用更详细的输出
```

## 关于实现的说明

Run a single test by specifying the `model` and test `case` as command line options. For example, to test flying a tailsitter in a mission you might run:

```bash
test/mavsdk_tests/mavsdk_test_runner.py test/mavsdk_tests/configs/sitl.json --speed-factor 10 --model tailsitter --case 'Fly square Multicopter Missions including RTL'
```

The easiest way to find out the current set of models and their associated test cases is to run all PX4 tests [as shown above](#run-all-px4-tests) (note, you can then cancel the build if you wish to test just one).

At time of writing the list generated by running all tests is:
```
About to run 39 test cases for 3 selected models (1 iteration):
  - iris:
    - 'Land on GPS lost during mission (baro height mode)'
    - 'Land on GPS lost during mission (GPS height mode)'
    - 'Continue on mag lost during mission'
    - 'Continue on baro lost during mission (baro height mode)'
    - 'Continue on baro lost during mission (GPS height mode)'
    - 'Continue on baro stuck during mission (baro height mode)'
    - 'Continue on baro stuck during mission (GPS height mode)'
    - 'Takeoff and Land'
    - 'Fly square Multicopter Missions including RTL'
    - 'Fly square Multicopter Missions with manual RTL'
    - 'Fly straight Multicopter Mission'
    - 'Offboard takeoff and land'
    - 'Offboard position control'
    - 'Fly forward in position control'
    - 'Fly forward in altitude control'
  - standard_vtol:
    - 'Land on GPS lost during mission (baro height mode)'
    - 'Land on GPS lost during mission (GPS height mode)'
    - 'Continue on mag lost during mission'
    - 'Continue on baro lost during mission (baro height mode)'
    - 'Continue on baro lost during mission (GPS height mode)'
    - 'Continue on baro stuck during mission (baro height mode)'
    - 'Continue on baro stuck during mission (GPS height mode)'
    - 'Takeoff and Land'
    - 'Fly square Multicopter Missions including RTL'
    - 'Fly square Multicopter Missions with manual RTL'
    - 'Fly forward in position control'
    - 'Fly forward in altitude control'
  - tailsitter:
    - 'Land on GPS lost during mission (baro height mode)'
    - 'Land on GPS lost during mission (GPS height mode)'
    - 'Continue on mag lost during mission'
    - 'Continue on baro lost during mission (baro height mode)'
    - 'Continue on baro lost during mission (GPS height mode)'
    - 'Continue on baro stuck during mission (baro height mode)'
    - 'Continue on baro stuck during mission (GPS height mode)'
    - 'Takeoff and Land'
    - 'Fly square Multicopter Missions including RTL'
    - 'Fly square Multicopter Missions with manual RTL'
    - 'Fly forward in position control'
    - 'Fly forward in altitude control'
```


## Notes on implementation

- The tests are invoked from the test runner script [mavsdk_test_runner.py](https://github.com/PX4/PX4-Autopilot/blob/master/test/mavsdk_tests/mavsdk_test_runner.py), which is written in Python.

  In addition to MAVSDK, this runner starts `px4` as well as Gazebo for SITL tests, and collects the logs of these processes.
- The test runner is a C++ binary that contains:
  - The [main](https://github.com/PX4/PX4-Autopilot/blob/master/test/mavsdk_tests/test_main.cpp) function to parse the arguments.
  - An abstraction around MAVSDK called [autopilot_tester](https://github.com/PX4/PX4-Autopilot/blob/master/test/mavsdk_tests/autopilot_tester.h).
  - The actual tests using the abstraction around MAVSDK as e.g. [test_multicopter_mission.cpp](https://github.com/PX4/PX4-Autopilot/blob/master/test/mavsdk_tests/test_multicopter_mission.cpp).
  - The tests use the [catch2](https://github.com/catchorg/Catch2) unit testing framework. The reasons for using this framework are:
      - Asserts (`REQUIRE`) which are needed to abort a test can be inside of functions (and not just in the top level test as is [the case with gtest](https://github.com/google/googletest/blob/master/docs/advanced.md#assertion-placement)).
      - Dependency management is easier because *catch2* can just be included as a header-only library.
      - *Catch2* supports [tags](https://github.com/catchorg/Catch2/blob/master/docs/test-cases-and-sections.md#tags), which allows for flexible composition of tests.


Terms used:
- "model": This is the selected Gazebo model, e.g. `iris`.
- "test case": This is a [catch2 test case](https://github.com/catchorg/Catch2/blob/master/docs/test-cases-and-sections.md).