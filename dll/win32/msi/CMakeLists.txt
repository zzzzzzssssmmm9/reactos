
remove_definitions(-D_WIN32_WINNT=0x502)
add_definitions(-D_WIN32_WINNT=0x600)

include_directories(${REACTOS_SOURCE_DIR}/sdk/include/reactos/wine)
add_definitions(-D__WINESRC__ -D__ROS_LONG64__ -DMSIRUNMODE=MSIRUNMODE_T)
spec2def(msi.dll msi.spec ADD_IMPORTLIB)
generate_idl_iids(msiserver.idl)

list(APPEND SOURCE
    action.c
    alter.c
    appsearch.c
    assembly.c
    automation.c
    classes.c
    create.c
    custom.c
    database.c
    delete.c
    dialog.c
    distinct.c
    drop.c
    files.c
    font.c
    format.c
    handle.c
    insert.c
    install.c
    media.c
    msi.c
    msi_main.c
    msiquery.c
    package.c
    patch.c
    record.c
    registry.c
    script.c
    select.c
    source.c
    storages.c
    streams.c
    string.c
    suminfo.c
    table.c
    tokenize.c
    update.c
    upgrade.c
    where.c)

BISON_TARGET(cond cond.y ${CMAKE_CURRENT_BINARY_DIR}/cond.tab.c COMPILE_FLAGS "-p cond_")
BISON_TARGET(sql sql.y ${CMAKE_CURRENT_BINARY_DIR}/sql.tab.c COMPILE_FLAGS "-p sql_")

list(APPEND PCH_SKIP_SOURCE
    ${BISON_cond_OUTPUTS}
    ${BISON_sql_OUTPUTS}
    ${CMAKE_CURRENT_BINARY_DIR}/msiserver_i.c
    ${CMAKE_CURRENT_BINARY_DIR}/msi_stubs.c)

add_library(msi MODULE
    ${SOURCE}
    ${PCH_SKIP_SOURCE}
    msi.rc
    ${CMAKE_CURRENT_BINARY_DIR}/msi.def)

if(MSVC)
    # Disable warning C4090: 'function': different 'const' qualifiers
    # Disable warning C4133: 'function': incompatible types - from 'UINT *' to 'MSIINSTALLCONTEXT *'
    # Disable warning C4146: unary minus operator applied to unsigned type, result still unsigned
    # Disable warning C4312: 'type cast': conversion from 'unsigned int' to 'HANDLE' of greater size
    target_compile_options(msi PRIVATE /wd4090 /wd4133 /wd4146 /wd4312)
endif()

add_idl_headers(msi_idlheader msiserver.idl)
add_typelib(msiserver.idl)
set_source_files_properties(msi.rc PROPERTIES OBJECT_DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/msiserver.tlb)
set_module_type(msi win32dll)
target_link_libraries(msi uuid ${PSEH_LIB} wine)
add_dependencies(msi msi_idlheader)
add_delay_importlibs(msi odbccp32 crypt32 wintrust)
add_importlibs(msi advapi32 advapi32_vista cabinet comctl32 gdi32 ole32 oleaut32 shell32 shlwapi urlmon user32 version wininet mspatcha
    #FIXME : should be in delayed imports
    imagehlp
    msvcrt
    kernel32_vista
    kernel32
    ntdll)

add_pch(msi precomp.h "${PCH_SKIP_SOURCE}")
add_cd_file(TARGET msi DESTINATION reactos/system32 FOR all)
