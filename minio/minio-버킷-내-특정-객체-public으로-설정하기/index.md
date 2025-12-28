---
title: "MinIO 버킷 내 특정 객체 Public으로 설정하기"
date: "2025-09-18"
keywords:
  [
    "MinIO",
    "미니오",
    "MinIO 버킷",
    "MinIO 접근 권한",
    "MinIO private",
    "MinIO public",
  ]
description: MinIO 버킷 내 특정 객체 Public으로 설정하는 방법
summary: MinIO에서 버킷 단위로 접근 권한을 Private, Public으로 설정하는 방법은 간단합니다. 해당 버킷 설정으로 들어가셔서 접근 권한을 선택해주시면 됩니다. 그러나 경우에 따라서는 Private 버킷 내에서 Public 객체를 만들고 싶을 수도 있습니다. GUI 환경에서는 제가 못 찾는 것인지 이러한 정책을 설정할 수 없는 것 같습니다. mc 명령어를 사용하면 간단하게 해결할 수 있습니다.
thumbnail: ./thumb.png
---

# MinIO 버킷 내 특정 객체 Public으로 설정하기

`MinIO`에서 버킷 단위로 접근 권한을 `Private`, `Public`으로 설정하는 방법은 간단합니다. 해당 버킷 설정으로 들어가셔서 접근 권한을 선택해주시면 됩니다.

그러나 경우에 따라서는 `Private` 버킷 내에서 `Public` 객체를 만들고 싶을 수도 있습니다. `GUI` 환경에서는 제가 못 찾는 것인지 이러한 정책을 설정할 수 없는 것 같습니다. `mc` 명령어를 사용하면 간단하게 해결할 수 있습니다.

## Public 변경 명령어

```text title="command"
mc anonymous set download <ALIAS>/<BUCKET>/<OBJECT>
```

예를들어, `myminio` alias의 `dev` 버킷에 있는 `documents/term.pdf` 파일을 `Public`으로 변경하고 싶다면 아래 명령어를 사용하여 접근 권한을 변경해주시면 됩니다.

```text title="command"
mc anonymous set download myminio/dev/documents/term.pdf
```
