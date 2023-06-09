message("run in ijkplayer")

#
# Copyright (c) 2019 Befovy <befovy@gmail.com>
#
# This file is part of fijkPlayer.
#
# fijkPlayer is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
#
# fijkPlayer is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with ijkPlayer; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA
#

if (ANDROID)
    if (${ANDROID_ABI} STREQUAL "armeabi-v7a")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfloat-abi=soft")
    endif ()
endif ()


set(BUILD_SOURCES
        ff_fferror.h
        ff_ffinc.h
        ff_ffmsg.h
        ff_ffmsg_queue.h

        ff_cmdutils.c
        ff_cmdutils.h
        ff_ffplay.c
        ff_ffplay.h
        ff_ffplay_debug.h
        ff_ffplay_def.h
        ff_ffplay_options.h

        ff_ffpipeline.c
        ff_ffpipeline.h
        ff_ffpipenode.c
        ff_ffpipenode.h
        ijkmeta.c
        ijkmeta.h

        ijkplayer.c
        ijkplayer.h
        ijkplayer_internal.h

        pipeline/ffpipeline_ffplay.c
        pipeline/ffpipeline_ffplay.h
        pipeline/ffpipenode_ffplay_vdec.c
        pipeline/ffpipenode_ffplay_vdec.h

        ijkavformat/allformats.c
        ijkavformat/ijklivehook.c
        ijkavformat/ijkmediadatasource.c
        ijkavformat/ijkio.c
        ijkavformat/ijkiomanager.c
        ijkavformat/ijkiocache.c
        ijkavformat/ijkioffio.c
        ijkavformat/ijkioprotocol.c
        ijkavformat/ijkioapplication.c
        ijkavformat/ijkiourlhook.c

        ijkavformat/ijkasync.c
        ijkavformat/ijkurlhook.c
        ijkavformat/ijklongurl.c
        ijkavformat/ijksegment.c

        ijkavutil/ijkdict.c
        ijkavutil/ijkutils.c
        ijkavutil/ijkthreadpool.c
        ijkavutil/ijktree.c
        ijkavutil/ijkfifo.c
        ijkavutil/ijkstl.cpp
        )

set(ANDROID_IJKPLAYER_SOURCES

        ijkavformat/ijkioandroidio.c

        android/ffmpeg_api_jni.c
        android/ijkplayer_android.c
        android/ijkplayer_jni.c

        android/pipeline/ffpipeline_android.c
        android/pipeline/ffpipenode_android_mediacodec_vdec.c
        )


if (ANDROID)
    set(BUILD_SOURCES ${BUILD_SOURCES} ${ANDROID_IJKPLAYER_SOURCES})
endif ()


add_library(ijkplayer SHARED ${BUILD_SOURCES})


target_include_directories(ijkplayer PRIVATE ${CMAKE_CURRENT_LIST_DIR})
target_include_directories(ijkplayer PRIVATE ${CMAKE_CURRENT_LIST_DIR}/..)
target_include_directories(ijkplayer PRIVATE ${FFMPAG_SHARED_DIR}/include)
target_include_directories(ijkplayer PRIVATE ijkavformat)
target_include_directories(ijkplayer PRIVATE ijkavutil)
target_include_directories(ijkplayer PRIVATE pipeline)
target_include_directories(ijkplayer PRIVATE ${CMAKE_CURRENT_LIST_DIR}/../ijkj4a)
target_include_directories(ijkplayer PRIVATE ${CMAKE_CURRENT_LIST_DIR}/../ijkyuv/include)

target_link_libraries(ijkplayer ijksdl)

if (ANDROID)
    target_link_libraries(ijkplayer ijksoundtouch)
    target_link_libraries(ijkplayer ijkffmpeg)
    target_link_libraries(ijkplayer android-ndk-profiler)
    target_link_libraries(ijkplayer log)
    target_link_libraries(ijkplayer android)
    target_link_libraries(ijkplayer jnigraphics)

    set(contrib_build_path "${ANROID_PROJECT_PATH}/../contrib/build")

    if (${ABI} STREQUAL "arm64-v8a")
        set(FFMPEG_INCLUDE
                "${contrib_build_path}/ffmpeg-arm64"
                )
    elseif (${ABI} STREQUAL "armeabi-v7a")
        set(FFMPEG_INCLUDE "${contrib_build_path}/ffmpeg-armv7a")
    else ()
        set(FFMPEG_INCLUDE "${contrib_build_path}/ffmpeg-${ABI}")
    endif ()

    include_directories("${FFMPEG_INCLUDE}/output/include")

endif ()


add_custom_command(TARGET ijkplayer
        PRE_BUILD
        COMMAND bash ${CMAKE_CURRENT_LIST_DIR}/version.sh ${CMAKE_CURRENT_LIST_DIR} ijkversion.h
        )