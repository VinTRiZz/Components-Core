# Core of Components system

## It's service repository, heart of components system. Required by every Components repository, that is not an other service repository

### Configurable options
1. (property) COMPONENTS_COMPONENT_ROOT_DIR -- Means directory where all Components-* placed. Used as a root to link targets. DEFAULT: ${CMAKE_CURRENT_SOURCE_DIR}/..
2. (property) COMPONENTS_INSTALL_PATH -- Path to install components libraries into. DEFAULT: ${CMAKE_CURRENT_SOURCE_DIR}/../BIN
3. (option) COMPONENTS_IS_ENABLED_QT -- Boolean property to define if every component uses Qt while building. Do not set ON if you develop any target without Qt and use Qt component (it may cause big troubles like SIGSEGV)

## Example of usage

```cmake
COMPONENTS_REQUIRE_QT(5 Core Gui Widgets)

COMPONENTS_CONFIGURE_COMPONENT(MyTarget
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    boost_system
    ssl
)
COMPONENTS_LINK_COMPONENT(MyTarget Logger)
```