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


--

        //Tummy
        Debug.DrawRay(transform.position, transform.rotation * Vector3.forward * 10f, Color.magenta);
        Debug.Log("Tummy" + Vector3.Dot(Vector3.down, transform.rotation * Vector3.forward));

        //mouth
        Debug.DrawRay(CatMouth.transform.position, CatMouth.transform.transform.rotation * Vector3.left * 10f, Color.magenta);
        Debug.Log("Mouth" + Vector3.Dot(Vector3.forward, CatMouth.transform.transform.rotation * Vector3.left));

        SetReward(0.002f * (Vector3.Dot(Vector3.down, transform.rotation * Vector3.forward)));
        SetReward(0.002f * (Vector3.Dot(Vector3.forward, CatMouth.transform.transform.rotation * Vector3.left)));
        SetReward(0.003f * (Head.localPosition.y - Head.localPosition.y));
        SetReward(0.001f); // Time Penalty

