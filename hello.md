# hello git

## git 명령어 요약

- clone: 원격 저장소 복사
- add: 스테이지 영역에 작업 파일 추가
- commit: 세이브, 스테이지 영역의 파일들을 가지고 커밋(=세이브)를 만들 수 있다.
- push: 원격 저장소에 커밋을 업로드한다.

## 브랜치 변경하기

- 브랜치란: 기존 내용을 유지한 채 새로운 내용을 추가하고 싶을 때 사용한다.
- 체크아웃: 특정 브랜치(혹은 커밋) 으로 돌아가고 싶을 때 사용.
- 소스트리의 체크아웃: 브랜치 이름을 더블 클릭하는것 만으로 체크아웃 가능.


        Vector3 eyeDir = Quaternion.Euler(Vector3.up * catEye.rotation.eulerAngles.y) * Vector3.forward;
        Vector3 targetToCatDir = Quaternion.LookRotation(targetTr.position - new Vector3(catEye.position.x, targetTr.position.y, catEye.position.z)) * Vector3.forward * 10f;

        Debug.DrawRay(catEye.position, eyeDir * 5f, Color.magenta);
        Debug.DrawRay(targetTr.position, targetToCatDir * 10f, Color.magenta);
