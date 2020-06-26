cmake_minimum_required (VERSION 3.6)

# --- Change the project's name here ---
project (project-ffi)
# --------------------------------------


## Package configuration ##
#  Add a 'private' subdirectory for racket source-code:
set(RKT_ADD_PRIVATE_SUBDIR NO CACHE BOOL "Enable an independent package for scribble documentation")

#  Have a separate package for docs: yes
set(RKT_ADD_SEPARATE_DOC_PACKAGE NO CACHE BOOL "Enable an independent package for scribble documentation")

#  Have a separate package for docs: yes
set(RKT_ADD_C_SRC_DIR NO CACHE BOOL "create a stub for a c project")

## make project directory
if(EXISTS ${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME})
    message(STATUS "Racket project exists")
    else()
    execute_process(COMMAND raco pkg new ${PROJECT_NAME}
                    WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
endif()

if(RKT_ADD_SEPARATE_DOC_PACKAGE)
    if(EXISTS ${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}-doc)
        message(STATUS "Racket doc project exists")   
    else()
        execute_process(COMMAND raco pkg new ${PROJECT_NAME}-doc
                        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
    endif()
endif()

if(RKT_ADD_PRIVATE_SUBDIR)
    if(EXISTS ${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}/private)
        message(STATUS ${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}/private " exists")
    else()
        file(MAKE_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}/private")
    endif()
endif()

#project custom target
file(GLOB rkt_src_files "${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}/*.rkt") 
    message(STATUS "Adding ${rkt_src_files} to rkt project")
if(RKT_ADD_PRIVATE_SUBDIR)
    file(GLOB rkt_src_private_files "${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}/private/*.rkt")
endif()

add_custom_target(racket_src SOURCES ${rkt_src_files} ${rkt_src_private_files} )

#docs custom target
if(RKT_ADD_SEPARATE_DOC_PACKAGE)
    file(GLOB rkt_doc_files "${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}-doc/scribblings/*.scrbl")
    #create a project-internals.scrbl
    message(STATUS "Creating scrbl internals")
    file(WRITE ${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}-doc/scribblings/${PROJECT_NAME}.scrbl
    #lang scribble/manual
    "@(require scribble/manual\n"
    "          scribble/basic\n"
    "          scribble/extract\n"
    "(for-label ${PROJECT_NAME}))\n"
    "\n"
    "@defmodule[${PROJECT_NAME}]\n"
    "\n"
    "@section{${PROJECT_NAME}'s Internal Documentation}\n"
    "\n"
    ";autogenererated list of files:\n"
    "\n"
    "@include-extracted[${rkt_src_files}]\n") 
    
else()
    file(GLOB rkt_doc_files "${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}/scribblings/*.scrbl")
endif()

add_custom_target(racket_docs SOURCES ${rkt_doc_files} )

if(RKT_ADD_C_SRC_DIR)
    #  Add src subdirectory
    if(EXISTS ${CMAKE_CURRENT_LIST_DIR}/src)
        message(STATUS ${CMAKE_CURRENT_LIST_DIR}/src " exists")

        else()
        
        #make directory
        file(MAKE_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/src")
        #write basic cpp, h file and cmake file:

        string(REPLACE "-" "_" project_name_uppercase ${PROJECT_NAME})
        string(TOUPPER ${project_name_uppercase} project_name_uppercase)

        file(WRITE ${CMAKE_CURRENT_LIST_DIR}/src/${PROJECT_NAME}.h
        "#ifndef ${project_name_uppercase}_H\n"
        "#define ${project_name_uppercase}_H\n\n"
        "#ifdef WIN32\n"
        "#define LIB_EXPORT __declspec(dllexport)\n"
        "#else\n"
        "#define LIB_EXPORT\n"
        "#endif\n"
        "extern \"C\" \{\n\n\} // extern C\n\n"
        "\#endif\n"
        )

        file(WRITE ${CMAKE_CURRENT_LIST_DIR}/src/${PROJECT_NAME}.cpp
        "#include \"${PROJECT_NAME}.h\"\n"
        "extern \"C\" \{\n
            LIB_EXPORT int addtwo(int a, int b) { return a+b;}
            \n\}\n\n"
        )
    endif()

    SET(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}")

    #  Add src cmake file and basic cpp files
    if(EXISTS ${CMAKE_CURRENT_LIST_DIR}/src/CMakeLists.txt)
        message(STATUS ${CMAKE_CURRENT_LIST_DIR}/src/CMakeLists.txt " exists")
        else()
        file(WRITE ${CMAKE_CURRENT_LIST_DIR}/src/CMakeLists.txt 
        "file(GLOB lib_${PROJECT_NAME}_src *.cpp *.h)\n"
        "add_library(lib-${PROJECT_NAME} SHARED \$\{lib_${PROJECT_NAME}_src\})\n"
        "install(TARGETS lib-${PROJECT_NAME} DESTINATION lib)"
        )
        
    endif()

    #add c sources
    add_subdirectory(src)

endif()