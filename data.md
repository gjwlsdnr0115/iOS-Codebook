# Data Persistence

## File System
App Sandbox
- Bundle container
- Data container
    - 앱에서 데이터를 저장하면 data container에 저장
- iCloud container

## Data Container
- Documents: 사용자가 직접 생성한 데이터 저장
- Library: 사용자가 직접 생성하지 않은 데이터 중에서 영구적으로 저장되어야 하는 데이터
    - Caches
    - Application Support: 설정 파일과 같은 앱실행에 필요한 파일 저장
- tmp: 특정 작업을 수행할 때 사용하는 임시 파일 저장

- Documents, Caches를 제외한 Library의 파일들은 자동으로 백업 된다
- 이때, 사용자가 직접 생성하지 않은 파일, 다운로드한 파일, 다시 생성 가능한 파일, 캐시 파일은 반드시 백업에서 제외 되어야 한다
- 안그러면 Reject!