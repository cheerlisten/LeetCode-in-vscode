# LeetCode-in-VSCode

```bash
git pull git@github.com:September007/LeetCode-in-vscode.git
cd LeetCode-in-vscode
git submodule update --init --recursive
```

## Config

### LeetCode problem
硬编码你的 leetcode.workspaceFolder 到custom-${SYSTEM}.cmake
示例：
**`vscode settings.json`**
```json
// 
{
    //...
    "leetcode.defaultLanguage": "cpp",
    "leetcode.hideSolved": true,
    "leetcode.endpoint": "leetcode-cn",
    "leetcode.workspaceFolder": "D:/CODE/LeetCode-in-vscode",
    "clang-format.style": "Chromium"
}
``` 
对应 **`${CMAKE_CURRENT_LIST_DIR}/custom-Windows.cmake`**
```cmake
set(LC_SOURCE_DIR "D:/CODE/LeetCode-in-vscode;${CMAKE_SOURCE_DIR}/ltct")
```
当然你可以指定多个目录，比如ltct用于竞赛题目

### main() type OJ
对于类似于洛谷的
添加配置
```cmake
set(LC_SOURCERAW_DIR ${CMAKE_SOURCE_DIR}/ot)
```


## Debug
gtest源文件 在embeded文件夹下出现，最终会添加相应目标，直接调试即可


## utils 库的使用


## tips

### cmake commands
*in vscode*: 
after install the [cmake-tools plugins](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) then you have multi commands to use
invoke the commands selector by { ctrl+shift+p then type 'cmake' }|{ctrl+p then type '>cmake' } 
* Delete Cache and Reconfigure
* Set Build Target
* Set Debug Target

you can trigger Cmake to configure by save (by ctrl+s when focus on CMakeLists.txt), if you can not, you can try this method show above
