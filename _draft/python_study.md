### Python study draft page
---
1. 특정 확장자 또는 형식의 file list 가져 오기
    - os module 과 glob module을 이용하여 가져올 수 있다.  
        - os module 사용 예
        ```python
        import os
        
        search_path = "./"
        file_list = os.listdir(search_path)
        file_list_cpp = [file for file in file_list if file.endswith(".cpp")]
        print(file_list_cpp)
        # print 결과 파일의 이름만 출력 한다.
        # ['test1.cpp', 'test2.cpp']
        ```
        - glob module 사용 예
        ```python
        import glob

        search_path = "./**/*"
        file_list = glob.glob(search_path, recursive=True)
        file_list_cpp = [file for file in file_list if file.endswith(".cpp")]
        print(file_list_cpp)
        # print 결과 파일의 path까지 출력 한다.
        # ['./test1.cpp', './path/test2.cpp']
        ```
        차이점:   
            - search_path 정의부분의 *의 사용 가능 여부  
            - recursive 한 search 가능 여부(os module에서 방법을 아직 못찾은 것 일수도...)


