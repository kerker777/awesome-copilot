---
description: 'C++ 專案配置和套件管理'
applyTo: '**/*.cmake, **/CMakeLists.txt, **/*.cpp, **/*.h, **/*.hpp'
---

此專案在清單模式下使用 vcpkg。在提供 vcpkg 建議時請記住這一點。不要提供像 vcpkg install library 這樣的建議,因為它們不會按預期工作。
如果可能,偏好透過 CMakePresets.json 設定快取變數和其他類型的東西。
提供有關可能影響建議或提及的 CMake 變數的任何 CMake 政策的資訊。
此專案需要跨平台和跨編譯器,支援 MSVC、Clang 和 GCC。
在提供使用檔案系統讀取文件的 OpenCV 範例時,請始終使用絕對文件路徑而不是文件名稱或相對文件路徑。例如,使用 `video.open("C:/project/file.mp4")`,而不是 `video.open("file.mp4")`。
