set(target my_i3)

file(GLOB my_i3_src "*.cc")

add_executable(my_i3 ${my_i3_src})
# target_link_libraries(my_i3 PRIVATE Boost::program_options)
target_link_libraries(my_i3 PUBLIC Threads::Threads PkgConfig::PKG_X11 PkgConfig::PKG_XTST)
