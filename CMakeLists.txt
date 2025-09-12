cmake_minimum_required(VERSION 3.10)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})  # Исправлено: PROJECT_VERSION вместо PROJECT_VESRION

project(helloworld VERSION ${PROJECT_VERSION})

# Поиск Boost с современным синтаксисом
find_package(Boost REQUIRED COMPONENTS unit_test_framework)

configure_file(version.h.in version.h)

add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
add_executable(test_version test_version.cpp)

# Установка стандарта C++14 для всех таргетов
set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

# Директории include
target_include_directories(helloworld
    PRIVATE 
        "${CMAKE_BINARY_DIR}"
        "${Boost_INCLUDE_DIRS}"  # Добавлено для Boost
)

# Директории include для test_version
target_include_directories(test_version
    PRIVATE 
        "${CMAKE_BINARY_DIR}"
        "${Boost_INCLUDE_DIRS}"
)

# Связывание библиотек с современным синтаксисом
target_link_libraries(helloworld_cli PRIVATE helloworld)

target_link_libraries(test_version PRIVATE
    helloworld
    Boost::unit_test_framework  # Современный синтаксис для Boost
)

# Компиляционные флаги
if (MSVC)
    target_compile_options(helloworld_cli PRIVATE /W4)
    target_compile_options(helloworld PRIVATE /W4)
    target_compile_options(test_version PRIVATE /W4)
else ()
    target_compile_options(helloworld_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(test_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()

# Установка
install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

# Настройка CPack для разных платформ
if(UNIX AND NOT APPLE)
    set(CPACK_GENERATOR "DEB")
elseif(APPLE)
    set(CPACK_GENERATOR "TGZ")  # Для macOS добавляем TGZ
elseif(WIN32)
    set(CPACK_GENERATOR "ZIP")
endif()

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT "example@example.com")
set(CPACK_PACKAGE_NAME "helloworld")

include(CPack)

# Тестирование
enable_testing()
add_test(NAME test_version COMMAND test_version)