# Еще немного об инструментах разработчика

## Введение в CMake

CMake - кроcсплатформенная утилита для автоматической сборки программы из исходного кода

CMake не занимается непосредственно сборкой, а лишь генерирует файлы сборки из предварительно написанного файла сценария CMakeLists.txt и предоставляет простой единый интерфейс управления
Файл CMakeLists.txt выглядит так:

```bash
cmake_minimum_required(VERSION 2.8.11 FATAL_ERROR)

# use LLVM/clang
#set(CMAKE_C_COMPILER clang)
#set(CMAKE_CXX_COMPILER clang++)

project(http_get) # Название проекта

include_directories(src) # Какие папки включить в проект

# envoirenment variables
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -pedantic")
# Здесь ${CMAKE_CXX_FLAGS} подставит флаги, добавленные пользователем вручную

# Здесь первым параметром идёт имя проекта,
а следующие параметры - это файлы, от которых зависит исполняемый файл
add_executable(http_get src/main.cpp src/TCPSocket.cpp src/Socket.cpp)
```

После создания файла `CMakeLists.txt` рядом с им обычно создаётся папка `build`. Создаём её вручную, переходим в неё и выполняем команду `cmake ../CMakeLists.txt`. После выполнения этой команды появятся папка с логами - `CMakeFiles`, а также `Makefile` - автоматически сгенерированный файл. Запускаем команду `make`. При изменении файла нужно снова запускать `make`, а не `cmake`.

## Coverage(Покрытие кода)

Coverage - покрытие кода, другими словами, сколько строк прошла программа.

Пример `CMakeLists.txt` с покрытием

```bash
 cmake_minimum_required(VERSION 2.8.11 FATAL_ERROR)

 set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_BINARY_DIR}/CMakeModules)
 file(DOWNLOAD
  https://raw.githubusercontent.com/bilke/cmake-modules/master/CodeCoverage.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/CMakeModules/CodeCoverage.cmake)

 project(optimalprimer)

 include_directories(src)

 # envoirenment variables
 set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -pedantic")  # $

 include(CodeCoverage)
 append_coverage_compiler_flags()

 add_executable(optimalprimer src/main.cpp src/bublegum.cpp)

 setup_target_for_coverage_lcov(
  NAME           coverage
  EXECUTABLE     optimalprimer
  DEPENDENCIES   optimalprimer
 )
```

Альтернативный вариант `CMakeLists.txt`:

```bash
cmake_minimum_required(VERSION 2.8.11 FATAL_ERROR)

project(optimalprimer)

include_directories(src)

# envoirenment variables
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -pedantic")
set(CMAKE_CXX_FLAGS "-g -O0 -fprofile-arcs -ftest-coverage")

add_executable(optimalprimer src/main.cpp src/bublegum.cpp)

list(APPEND COV_GCNO_GCDA ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/optimalprimer.dir/src/main.cpp.gcno)
list(APPEND COV_GCNO_GCDA ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/optimalprimer.dir/src/main.cpp.gcda)
list(APPEND COV_GCNO_GCDA ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/optimalprimer.dir/src/bublegum.cpp.gcno)
list(APPEND COV_GCNO_GCDA ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/optimalprimer.dir/src/bublegum.cpp.gcda)

add_custom_target(coverage
    COMMAND mkdir -p coverage
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
add_custom_command(TARGET coverage
    COMMAND optimalprimer
    COMMAND lcov -d . --capture --output-file coverage.info
    COMMAND genhtml -o coverage coverage.info
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
add_dependencies(coverage optimalprimer)
set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
    coverage
    coverage.info
    ${COV_GCNO_GCDA}
)
```

## GoogleTest

GoogleTest используется для unit-тестирования. Включает в себя, кроме тестов, также и gmock, который указывает, какие классы (методы) являются заглушкой и не учитывает тесты с заглушкой как проваленные. Его надо установить отдельно. После установки создадим папку tests рядом с папками src и build. В ней создадим первый файл с тестами test.cpp:

```cpp
#include <gtest/gtest.h>
TEST(FunnyTests, StrangeTest)
{
 ASSERT_EQ(2+2, 5) << "lol, wtf";
}

int main(int argc, char **argv)
{
::testing::InitGoogleTest(&argc, argv);
return RUN_ALL_TESTS();
}
```

Компиляция теста: `g++ test.cpp -lgtest`

Чтобы объединить GoogleTest и CMake лучше почитать документацию

## Анализ покрытия

Алгоритм получения отчёта о покрытии в HTML формате.

1. Задаём ключи при компиляции:

```bash
g++ -f profile-arcs -f test-coverage <file> -o <out_file> -lgcov
```

2. Появятся файлы `.gcno` - что может быть выполнено. Запускаем программу.
3. После запуска и завершения программы появляются файлы `.gcda` - что было выполнено
4. `lcov` - утилита для сбора gcno и gcda в один отчёт. Использование:

```bash
  lcov -d . --capture --output_file coverage.info
  # coverage.info - файл с отчётом, а флаг -d говорит откуда брать файлы gcno и gcda
```

5. Используем genhtml, чтобы сделать отчёт читаемым. Использование:

```bash
genhtml -o report coverage.info
# здесь report - папка с сайтом отчёта
```

Анализ покрытия можно автоматизировать при помощи CMake. Пример CMakeLists.txt в папке с тестами:

```bash
 # envoirenment variables
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -pedantic")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage")

add_executable(
    RLE
    main.cpp
    RLE.hpp
)

# Две команды ниже - команды для работы с механизмом списков в CMake.
# Первый аргумент - действие над списком
# Второй аргумент - название списка, над которым совершается действие

list(APPEND COV_GCNO_GCDA ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/RLE.dir/main.cpp.gcno)
list(APPEND COV_GCNO_GCDA ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/RLE.dir/main.cpp.gcda)
add_custom_target(coverage
    COMMAND mkdir -p coverage
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
add_custom_command(TARGET coverage
    COMMAND RLE
    COMMAND lcov --directory ./src --capture --output-file ./coverage.info
    COMMAND lcov --remove ./coverage.info -o filtered.info 'bits/*' 'googlemock/*' '/usr/*' 'src/main.cpp'
    COMMAND genhtml -o coverage filtered.info
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)
add_dependencies(coverage RLE)ы
set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
    coverage
    coverage.info
    ${COV_GCNO_GCDA} #$
)
```
