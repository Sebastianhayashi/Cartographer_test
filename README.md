# Cartographer 测试

## 旧环境准备

由于在 bianbuOS 上没有 python3-wstool 可用，并且如果只是使用 pip 去进行安装的话，会报错说依赖无法满足（原因是目前的环境太新，wstool 需要老环境）。具体错误如下：

```
cartographer) bianbu@k1:~$ pip install wstool
Collecting wstool
  Using cached wstool-0.1.17.tar.gz (53 kB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... error
  error: subprocess-exited-with-error

  × Getting requirements to build wheel did not run successfully.
  │ exit code: 1
  ╰─> [23 lines of output]
      Traceback (most recent call last):
        File "/home/bianbu/cartographer/lib/python3.13/site-packages/pip/_vendor/pyproject_hooks/_in_process/_in_process.py", line 389, in <module>
          main()
          ~~~~^^
        File "/home/bianbu/cartographer/lib/python3.13/site-packages/pip/_vendor/pyproject_hooks/_in_process/_in_process.py", line 373, in main
          json_out["return_val"] = hook(**hook_input["kwargs"])
                                   ~~~~^^^^^^^^^^^^^^^^^^^^^^^^
        File "/home/bianbu/cartographer/lib/python3.13/site-packages/pip/_vendor/pyproject_hooks/_in_process/_in_process.py", line 143, in get_requires_for_build_wheel
          return hook(config_settings)
        File "/tmp/pip-build-env-xzbuvipr/overlay/lib/python3.13/site-packages/setuptools/build_meta.py", line 331, in get_requires_for_build_wheel
          return self._get_build_requires(config_settings, requirements=[])
                 ~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "/tmp/pip-build-env-xzbuvipr/overlay/lib/python3.13/site-packages/setuptools/build_meta.py", line 301, in _get_build_requires
          self.run_setup()
          ~~~~~~~~~~~~~~^^
        File "/tmp/pip-build-env-xzbuvipr/overlay/lib/python3.13/site-packages/setuptools/build_meta.py", line 512, in run_setup
          super().run_setup(setup_script=setup_script)
          ~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "/tmp/pip-build-env-xzbuvipr/overlay/lib/python3.13/site-packages/setuptools/build_meta.py", line 317, in run_setup
          exec(code, locals())
          ~~~~^^^^^^^^^^^^^^^^
        File "<string>", line 7, in <module>
      ModuleNotFoundError: No module named 'imp'
      [end of output]

  note: This error originates from a subprocess, and is likely not a problem with pip.
error: subprocess-exited-with-error

× Getting requirements to build wheel did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.
(cartographer) bianbu@k1:~$
```

```
sudo apt update
sudo apt install -y \
  make build-essential libssl-dev zlib1g-dev libbz2-dev \
  libreadline-dev libsqlite3-dev wget curl llvm \
  libncurses5-dev libncursesw5-dev xz-utils tk-dev \
  libffi-dev liblzma-dev
```

安装 pyenv

```
curl https://pyenv.run | bash
```

安装完后，按提示把下面几行加到 ~/.bashrc（或 ~/.bash_profile）末尾：

```
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

然后重新加载一次 shell 配置：

```
source ~/.bashrc
```
 用 pyenv 安装兼容的 Python 版本（比如 3.11）：

```
pyenv install 3.11.0
pyenv virtualenv 3.11.0 carto311
pyenv activate carto311
```

最后确认一下版本，然后开始安装 wstool：

```
python -V 

pip install --upgrade pip
pip install wstool
```

> 需要激活环境的话，请使用指令 `pyenv activate carto311`

## 编译 cartographer

依赖安装：

```
sudo apt update
sudo apt install -y libboost-all-dev libcairo2-dev git cmake ninja-build g++ \
  libeigen3-dev \
  libceres-dev \
  libprotobuf-dev protobuf-compiler \
  libsuitesparse-dev \
  liblua5.3-dev \
  libabsl-dev 
```

开始编译：

```
cd ~
git clone https://github.com/cartographer-project/cartographer.git
cd cartographer
mkdir build && cd build

cmake .. -G Ninja \
  -DBUILD_TESTING=OFF \
  -DCARTOGRAPHER_BUILD_TESTING=OFF
ninja          # 编译
ninja test     # 可选：跑单元测试
sudo ninja install   # 可选：安装到 /usr/local
```



增加 `  -DBUILD_TESTING=OFF -DCARTOGRAPHER_BUILD_TESTING=OFF` 是因为会遇到如下的问题：

```
ng.h:50:8: error: expected ‘;’ at end of member declaration 50 | bool running_ GUARDED_BY(mutex_) = true; | ^~~~~~~~ | ; /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:50:28: error: ‘mutex_’ is not a type 50 | bool running_ GUARDED_BY(mutex_) = true; | ^~~~~~ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:50:42: error: invalid pure specifier (only ‘= 0’ is allowed) before ‘;’ token 50 | bool running_ GUARDED_BY(mutex_) = true; | ^ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:51:8: error: expected ‘;’ at end of member declaration 51 | bool idle_ GUARDED_BY(mutex_) = true; | ^~~~~ | ; /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:51:25: error: ‘mutex_’ is not a type 51 | bool idle_ GUARDED_BY(mutex_) = true; | ^~~~~~ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:51:39: error: invalid pure specifier (only ‘= 0’ is allowed) before ‘;’ token 51 | bool idle_ GUARDED_BY(mutex_) = true; | ^ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:51:14: error: ‘int cartographer::common::testing::ThreadPoolForTesting::GUARDED_BY(int)’ cannot be overloaded with ‘int cartographer::common::testing::ThreadPoolForTesting::GUARDED_BY(int)’ 51 | bool idle_ GUARDED_BY(mutex_) = true; | ^~~~~~~~~~ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:50:17: note: previous declaration ‘int cartographer::common::testing::ThreadPoolForTesting::GUARDED_BY(int)’ 50 | bool running_ GUARDED_BY(mutex_) = true; | ^~~~~~~~~~ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:52:37: error: expected ‘;’ at end of member declaration 52 | std::deque<std::shared_ptr<Task>> task_queue_ GUARDED_BY(mutex_); | ^~~~~~~~~~~ | ; /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:52:60: error: ‘mutex_’ is not a type 52 | std::deque<std::shared_ptr<Task>> task_queue_ GUARDED_BY(mutex_); | ^~~~~~ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:52:49: error: ‘int cartographer::common::testing::ThreadPoolForTesting::GUARDED_BY(int)’ cannot be overloaded with ‘int cartographer::common::testing::ThreadPoolForTesting::GUARDED_BY(int)’ 52 | std::deque<std::shared_ptr<Task>> task_queue_ GUARDED_BY(mutex_); | ^~~~~~~~~~ /home/bianbu/cartographer/cartographer/common/internal/testing/thread_pool_for_testing.h:50:17: note: previous declaration ‘int cartographer::common::testing::ThreadPoolForTesting::GUARDED_BY(int)’
```

但是目前编译还是会遇到 Abseil 相关问题，具体 log 如下：

```
/usr/include/absl/synchronization/mutex.h:726:3: note: candidate: ‘template<class T> absl::debian5::Condition::Condition(const T*, bool (absl::debian5::internal::identity<T>::type::*)() const)’
  726 |   Condition(const T* object,
      |   ^~~~~~~~~
/usr/include/absl/synchronization/mutex.h:726:3: note:   candidate expects 2 arguments, 1 provided
/usr/include/absl/synchronization/mutex.h:722:3: note: candidate: ‘template<class T> absl::debian5::Condition::Condition(T*, bool (absl::debian5::internal::identity<T>::type::*)())’
  722 |   Condition(T* object, bool (absl::internal::identity<T>::type::*method)());
      |   ^~~~~~~~~
/usr/include/absl/synchronization/mutex.h:722:3: note:   candidate expects 2 arguments, 1 provided
/usr/include/absl/synchronization/mutex.h:711:3: note: candidate: ‘template<class T, class> absl::debian5::Condition::Condition(bool (*)(T*), typename absl::debian5::internal::identity<T>::type*)’
  711 |   Condition(bool (*func)(T*), typename absl::internal::identity<T>::type* arg);
      |   ^~~~~~~~~
/usr/include/absl/synchronization/mutex.h:711:3: note:   candidate expects 2 arguments, 1 provided
/usr/include/absl/synchronization/mutex.h:697:3: note: candidate: ‘template<class T> absl::debian5::Condition::Condition(bool (*)(T*), T*)’
  697 |   Condition(bool (*func)(T*), T* arg);
```

> 由于 log 过长，全 log 请参考[这里](https://raw.githubusercontent.com/Sebastianhayashi/Cartographer_test/refs/heads/main/absl_log.txt)。