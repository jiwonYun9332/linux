## OverlayFS

OverlayFS는 통합 파일 시스템 유형이다. 이를 통해 사용자는 하나의 파일 시스템을 다른 파일 시스템 상단에 `overlay` 할 수 있다. 변경 사항은 상단 파일 시스템에 기록되는 반면
하단 파일 시스템은 변경되지 않은 상태로 남아 있게 된다. 이를 통해 여러 사용자는 컨테이너 또는 DVD-ROM과 같은 파일 시스템 이미지를 공유할 수 있다. 이때 기본 이미지는 
읽기 전용 미디어에 배치된다.

[Overlay](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt)

> **작동 방식**

- lower dir: 아래쪽에 위치한 디렉토리 레이어. 여기에 위치한 entry들은 read-only로 마운트되며, 만약 수정될 경우 upper dir에 COW로 적용되고
만약 삭제될 경우 특별히 삭제됨을 표시하는 whiteout이라는 특별한 파일을 통해서 삭제됨을 기록한다.
- upper dir: 최종적으로 write 권한이 있는 맨 최상위 레이어로써, 모든 수정사항 즉 삭제와 파일의 수정은 이 upper dir에 기록된다.
upper dir에서 마스킹하는 디렉토리는 하위 디렉토리에 위치해 있더라도 무시된다.
- merge dir: 모든 upper dir + lower dir이 표시되는 환경
- work dir: 통합 뷰의 원자성을 보장하기 위해 존재하는 중간 계층, Overlay를 직접 사용할 때는 크게 중요하지 않다.

다음 커맨드는 overlay fs를 생성하여 merge라는 새로운 디렉토리에 표시하고 있다.

```
mount -t overlay overlay -o lowerdir=lower1/,upperdir=upper/,workdir=work/ merge/
```

> **Docker에서의 이용**

OverlayFS를 통해서 docker는 docker image들은 수정되지 않으면서, 새로운 컨테이너에서 만든 변경사항과 삭제된 파일을 새로운 R/W 전용의 컨테이너 레이어 디렉토리에 따로 저장한다. 
이 두 레이어(lower dir: docker image, upper dir: container dir)을 더하여 merged dir 즉 사용자에게 보여지는 완전한 root directory를 보여준다. 새로운 도커 이미지를 생성해야 할
때는 컨테이너 레이어(upper)를 스냅샷으로 만든 뒤 새로운 이미지 레이어로서 추가하는 방식이다. 즉, 하나의 이미지에 여러 개의 이미지 레이어가 존재한다면 각 레이어들은 언젠가 한 번은 
upper 레이어였던 적이 있으며, 현재는 모두 lower 레이어로서 컨테이너에 읽기 전용으로 제공된다는 뜻이다.
