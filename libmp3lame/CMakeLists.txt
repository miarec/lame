set (CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE ON)

add_library(mp3lame)

list( APPEND SOURCES
   PRIVATE
      VbrTag.c
      bitstream.c
      encoder.c
      fft.c
      gain_analysis.c
      id3tag.c
      lame.c
      newmdct.c
      presets.c
      psymodel.c
      quantize.c
      quantize_pvt.c
      reservoir.c
      set_get.c
      tables.c
      takehiro.c
      util.c
      vbrquantize.c
      version.c
)


if (${WITH_XMM})
    list( APPEND SOURCES
        PRIVATE
            vector/xmm_quantize_sub.c
    )
endif()

target_sources(mp3lame PRIVATE ${SOURCES})

set_target_properties(mp3lame PROPERTIES
  POSITION_INDEPENDENT_CODE ON
)

target_include_directories(mp3lame
    PRIVATE 
        ${CMAKE_CURRENT_SOURCE_DIR}
    PUBLIC 
        # where top-level project will look for the library's public headers
        $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/../include>
        # where external projects will look for the library's public headers
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
