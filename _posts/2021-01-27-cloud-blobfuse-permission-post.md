---
title: "클라우드: blobfuse"
date: 2021-01-27 00:00:00
categories: cloud, storage
---

dir_mode와 file_mode를 사용하는 예제가 몇몇 커뮤니티에 올라와 있던데 사실이 아닙니다. blobfuse 코드 상으로 확인해봐도 해당 구현은 없으며 allow_other 옵션을 사용하는 경우에는 777, 그렇지 않은 경우에는 770으로 일괄 적용됩니다.

```cpp
int read_and_set_arguments(int argc, char *argv[], struct fuse_args *args)
{
    // FUSE has a standard method of argument parsing, here we just follow the pattern.
    *args = FUSE_ARGS_INIT(argc, argv);

    // Check for existence of allow_other flag and change the default permissions based on that
    config_options.defaultPermission = 0770;
    std::vector<std::string> string_args(argv, argv+argc);
    for (size_t i = 1; i < string_args.size(); ++i) {
      if (string_args[i].find("allow_other") != std::string::npos) {
          config_options.defaultPermission = 0777; 
      }
    }
    ...
}
```

blobfuse의 성능은 일정하지 않습니다. 마운트된 볼륨에서 파일 읽기 작업이 시도되면 임시 디렉터리에 blob 다운로드가 시작됩니다. 다운로드하는 중에는 파일 읽기 작업은 아직 시작되지 않습니다. 다운로드가 완료되면 그때서야 파일 읽기 작업을 시작할 수 있습니다. 임시 디렉터리의 만료 시간이 지나기 전까지는 한 번 다운로드했던 blob은 다운로드없이 읽기가 가능합니다.

blobfuse가 이런 방식으로 동작하다보니 blob의 크기가 클수록 파읽 읽기 작업이 가능하기까지의 지연 시간은 늘어날 수 밖에 없습니다. blobfuse를 사용하기 전에 이러한 설계 상의 한계는 고려해야 합니다.



