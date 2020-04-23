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

using UnityEngine;
using MLAgents;
using MLAgentsExamples;
using MLAgents.Sensors;

public class LandingAgent : Agent
{
    [Header("Body Parts")] 
    [Space(10)] 
    public Transform Tummy;
    public Transform BackBone_02;
    public Transform LegFrontRight_01; 
    public Transform LegFrontRight_02; 
    public Transform LegFrontLeft_01; 
    public Transform LegFrontLeft_02;
    public Transform BackBone_03;
    public Transform LegRearLeft_02; 
    public Transform LegRearLeft_03; 
    public Transform LegRearRight_02; 
    public Transform LegRearRight_03; 
    public Transform Head; 
    public Transform Tail_01; 
    public Transform Tail_02; 
    public Transform Tail_03; 
    public Transform Tail_04; 
    public Transform Tail_05;

    [Header("Etc Settings")]
    [Space(10)]
    public MeshRenderer groundRd;
    Color groundBasicColor;


    Vector3 dirToTarget;
    JointDriveController m_JdController;

    public override void Initialize()
    {
        //Number of Body Parts = 15
        m_JdController = GetComponent<JointDriveController>();
        m_JdController.SetupBodyPart(Tummy);
        m_JdController.SetupBodyPart(BackBone_02);
        m_JdController.SetupBodyPart(LegFrontRight_01);
        m_JdController.SetupBodyPart(LegFrontRight_02);
        m_JdController.SetupBodyPart(LegFrontLeft_01);
        m_JdController.SetupBodyPart(LegFrontLeft_02);
        m_JdController.SetupBodyPart(BackBone_03);
        m_JdController.SetupBodyPart(LegRearLeft_02);
        m_JdController.SetupBodyPart(LegRearLeft_03);
        m_JdController.SetupBodyPart(LegRearRight_02);
        m_JdController.SetupBodyPart(LegRearRight_03);
        m_JdController.SetupBodyPart(Head);
        m_JdController.SetupBodyPart(Tail_01);
        m_JdController.SetupBodyPart(Tail_02);
        m_JdController.SetupBodyPart(Tail_03);
        m_JdController.SetupBodyPart(Tail_04);
        m_JdController.SetupBodyPart(Tail_05);
        groundBasicColor = groundRd.material.color;
    }
    public void CollectObservationBodyPart(BodyPart bp, VectorSensor sensor)
    {
        // Number of Body Parts = n
        // Vector Observation's Space Size = (2n - (1 + a)) * 7 + etc
        var rb = bp.rb;
        // Whether the bp touching the ground
        // - Apply all except Bottom Leg
        sensor.AddObservation(bp.groundContact.touchingGround ? 1 : 0); // Whether the bp touching the ground
        sensor.AddObservation(rb.velocity);
        sensor.AddObservation(rb.angularVelocity);

        if (bp.rb.transform != Tummy && bp.rb.transform != Tail_01 && bp.rb.transform != Tail_02 &&
         bp.rb.transform != Tail_03 && bp.rb.transform != Tail_04 && bp.rb.transform != Tail_05)
        {
            var localPosRelToBody = Tummy.InverseTransformPoint(rb.transform.localPosition);
            sensor.AddObservation(localPosRelToBody);
            sensor.AddObservation(bp.currentXNormalizedRot); // Current x rot
            sensor.AddObservation(bp.currentYNormalizedRot); // Current y rot
            sensor.AddObservation(bp.currentZNormalizedRot); // Current z rot
            sensor.AddObservation(bp.currentStrength / m_JdController.maxJointForceLimit);
        }
    }

    public override void CollectObservations(VectorSensor sensor)
    {
        m_JdController.GetCurrentJointForces();
        sensor.AddObservation(Tummy.transform.localPosition);

        foreach (var bodyPart in m_JdController.bodyPartsDict.Values)
        {
            CollectObservationBodyPart(bodyPart, sensor);
        }

        sensor.AddObservation(Tummy.localPosition);
    }

    public override void OnActionReceived(float[] vectorAction)
    {
        // The dictionary with all the body parts in it are in the jdController
        var bpDict = m_JdController.bodyPartsDict;

        var i = -1;
        // Pick a new target joint rotation - 25
        //Back Bone - 6
        bpDict[BackBone_02].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], vectorAction[++i]);
        bpDict[BackBone_03].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], vectorAction[++i]);

        //Front Leg - 8
        bpDict[LegFrontRight_01].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);
        bpDict[LegFrontRight_02].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);
        bpDict[LegFrontLeft_01].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);
        bpDict[LegFrontLeft_02].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);

        //Back Leg - 8
        bpDict[LegRearLeft_02].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);
        bpDict[LegRearLeft_03].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);
        bpDict[LegRearRight_02].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);
        bpDict[LegRearRight_03].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], 0);

        //Head - 3
        bpDict[Head].SetJointTargetRotation(vectorAction[++i], vectorAction[++i], vectorAction[++i]);

        // Update joint strength - 11
        bpDict[BackBone_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegFrontRight_01].SetJointStrength(vectorAction[++i]);
        bpDict[LegFrontRight_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegFrontLeft_01].SetJointStrength(vectorAction[++i]);
        bpDict[LegFrontLeft_02].SetJointStrength(vectorAction[++i]);

        bpDict[BackBone_03].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearLeft_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearLeft_03].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearRight_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearRight_03].SetJointStrength(vectorAction[++i]);
        bpDict[Head].SetJointStrength(vectorAction[++i]);
        
        foreach (var bodyPart in m_JdController.bodyPartsDict.Values)
        {
            if(bodyPart.rb.transform == LegFrontRight_02 ||
            bodyPart.rb.transform == LegFrontLeft_02 ||
            bodyPart.rb.transform == LegRearLeft_03 ||
            bodyPart.rb.transform == LegRearRight_03 &&
            bodyPart.groundContact.touchingGround == true)
            SetReward(0.0001f);
        }

        SetReward(0.003f * (Head.localPosition.y - Head.localPosition.y));
        SetReward(0.001f); // Time Penalty
    }

    public override void OnEpisodeBegin()
    {
        foreach (var bodyPart in m_JdController.bodyPartsDict.Values)
        {
            bodyPart.Reset(bodyPart);
        }
        this.transform.localPosition = Vector3.up * (20f + Random.Range(-1f, 15f));

        Invoke("InitGroundColor", 0.5f);
    }

    void InitGroundColor()
    {
        groundRd.material.color = groundBasicColor;
    }

    public override void Heuristic(float[] action)
    {
        for (int i = 0; i < 36; i++)
        {
            action[i] = Random.Range(-1f, 1f);
        }
    }
}

