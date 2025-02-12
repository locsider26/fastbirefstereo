cmake_minimum_required(VERSION 3.10)

PROJECT(uneven_rgbd)

find_package(OpenCV REQUIRED COMPONENTS core highgui imgproc imgcodecs)

# Copy resources to binary folder
file(COPY "resources" DESTINATION "${CMAKE_CURRENT_BINARY_DIR}")

# Copy OpenCL program to binary folder
configure_file("src/bilateral_filter.cl" "${CMAKE_CURRENT_BINARY_DIR}" COPYONLY)

if(WIN32)
	# Copy OpenCV dlls to binary folder
	string(REGEX REPLACE "\\." "" OpenCV_VERSION_NAME ${OpenCV_VERSION})

	if(EXISTS ${OpenCV_DIR}/x64/vc15/bin/opencv_world${OpenCV_VERSION_NAME}.dll)
		file(COPY "${OpenCV_DIR}/x64/vc15/bin/opencv_world${OpenCV_VERSION_NAME}.dll" DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
		file(COPY "${OpenCV_DIR}/x64/vc15/bin/opencv_world${OpenCV_VERSION_NAME}d.dll" DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

	else(EXISTS ${OpenCV_DIR}/x64/vc15/bin/opencv_world${OpenCV_VERSION_NAME}.dll)
		foreach(lib ${OpenCV_LIBS})
			file(COPY "${OpenCV_DIR}/bin/Debug/${lib}${OpenCV_VERSION_NAME}d.dll" DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
			file(COPY "${OpenCV_DIR}/bin/Release/${lib}${OpenCV_VERSION_NAME}.dll" DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
		endforeach(lib)
	endif(EXISTS ${OpenCV_DIR}/x64/vc15/bin/opencv_world${OpenCV_VERSION_NAME}.dll)
endif(WIN32)

set(SRC_DEMO
	src/main_demo.cpp
	src/depth_estimator.cpp
	src/depth_estimator.h
	src/bilateral_filter.cl
)

set(SRC_RECTIFICATION
	src/main_rectification.cpp
	src/rectifier.cpp
	src/rectifier.h
)

add_executable(uneven_rgbd_demo ${SRC_DEMO})
target_link_libraries(uneven_rgbd_demo ${OpenCV_LIBS})

add_executable(precompute_rectification ${SRC_RECTIFICATION})
target_link_libraries(precompute_rectification ${OpenCV_LIBS})
