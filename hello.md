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

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using MLAgents;
using MLAgentsExamples;
using MLAgents.Sensors;

public class CatAgent : Agent
{
    [Header("Body Parts")]
    [Space(10)]
    public Transform Tummy;
    public Transform LegFrontRight_01;
    public Transform LegFrontRight_02;
    public Transform LegFrontLeft_01;
    public Transform LegFrontLeft_02;
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
    public Transform targetTr;
    public Rigidbody catHead;
    public MeshRenderer groundRd;
    public Transform catEye;
    Color groundBasicColor;


    Vector3 dirToTarget;
    Transform originCatTailTr;

    JointDriveController m_JdController;
    Matrix4x4 m_TargetDirMatrix;

    public override void Initialize()
    {
        //Number of Body Parts = 15
        m_JdController = GetComponent<JointDriveController>();
        m_JdController.SetupBodyPart(Tummy);
        m_JdController.SetupBodyPart(LegFrontRight_01);
        m_JdController.SetupBodyPart(LegFrontRight_02);
        m_JdController.SetupBodyPart(LegFrontLeft_01);
        m_JdController.SetupBodyPart(LegFrontLeft_02);
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
        
        eyeDir = Quaternion.Euler(Vector3.up * catEye.rotation.eulerAngles.y) * Vector3.forward;
        targetToCatDir = Quaternion.LookRotation
        (targetTr.position - new Vector3(catEye.position.x, targetTr.position.y, catEye.position.z))
         * Vector3.forward;
    }
    public void CollectObservationBodyPart(BodyPart bp, VectorSensor sensor)
    {
        // Number of Body Parts = n
        // Vector Observation's Space Size = (2n - (1 + a)) * 7 + etc
        var rb = bp.rb;
        // Whether the bp touching the ground
        // - Apply all except Bottom Leg
        sensor.AddObservation(bp.groundContact.touchingGround ? 1 : 0); // Whether the bp touching the ground

        var velocityRelativeToLookRotationToTarget = m_TargetDirMatrix.inverse.MultiplyVector(rb.velocity);
        sensor.AddObservation(velocityRelativeToLookRotationToTarget);

        var angularVelocityRelativeToLookRotationToTarget = m_TargetDirMatrix.inverse.MultiplyVector(rb.angularVelocity);
        sensor.AddObservation(angularVelocityRelativeToLookRotationToTarget);

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

        sensor.AddObservation(targetTr.localPosition);
        sensor.AddObservation(Tummy.localPosition);
        sensor.AddObservation(Vector3.Distance(Tummy.localPosition, targetTr.localPosition));
        sensor.AddObservation(eyeDir);
        sensor.AddObservation(targetToCatDir);
    }

    public override void OnActionReceived(float[] vectorAction)
    {
        // The dictionary with all the body parts in it are in the jdController
        var bpDict = m_JdController.bodyPartsDict;

        var i = -1;
        // Pick a new target joint rotation - 15
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

        // Update joint strength - 9
        bpDict[LegFrontRight_01].SetJointStrength(vectorAction[++i]);
        bpDict[LegFrontRight_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegFrontLeft_01].SetJointStrength(vectorAction[++i]);
        bpDict[LegFrontLeft_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearLeft_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearLeft_03].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearRight_02].SetJointStrength(vectorAction[++i]);
        bpDict[LegRearRight_03].SetJointStrength(vectorAction[++i]);
        bpDict[Head].SetJointStrength(vectorAction[++i]);

        RewardHeadDir();
        RewardDistanceTarget();
        SetReward(-0.001f); // Time Penalty
    }

    float preDistanceMag = 50f;
    Vector3 eyeDir;
    Vector3 targetToCatDir;

    void RewardHeadDir()
    {
        eyeDir = Quaternion.Euler(Vector3.up * catEye.rotation.eulerAngles.y) * Vector3.forward;
        targetToCatDir = Quaternion.LookRotation
        (targetTr.position - new Vector3(catEye.position.x, targetTr.position.y, catEye.position.z))
         * Vector3.forward;

        //Debug.DrawRay(catEye.position, eyeDir * 5f, Color.magenta);
        //Debug.DrawRay(targetTr.position, targetToCatDir * 10f, Color.magenta);

        SetReward(0.002f * Vector3.Dot(eyeDir, targetToCatDir)); 
    }

    void RewardDistanceTarget()
    {
        float currDistanceMag = Mathf.Abs(Vector3.Distance(Tummy.localPosition, targetTr.localPosition));
        if(preDistanceMag - currDistanceMag >= 0.02f) SetReward(0.001f);
        preDistanceMag = currDistanceMag;
    }

    void RewardHeadDirChange()
    {
        dirToTarget = targetTr.localPosition - catHead.transform.localPosition;
        float movingTowardsDot = Vector3.Dot(
            catHead.velocity,
            dirToTarget.normalized);
        SetReward(0.001f * movingTowardsDot);
    }

    public void TouchTarget()
    {
        SetReward(1f);
        groundRd.material.color = Color.green;
        EndEpisode();
    }

    public override void OnEpisodeBegin()
    {
        float targetX, targetZ;

        //Target 위치 랜덤 변경
        //targetTr.localPosition = new Vector3(Random.Range(-22.5f, 22.5f), 2f, Random.Range(-22.5f, 22.5f));
        
        targetX = Random.Range(-13f, 13f);
        targetZ = Random.Range(-13f, 13f);
        
        if(targetX >= 0) targetX += 2;
        else targetX -= 2;
        if(targetZ >= 0) targetZ += 2;
        else targetZ = targetZ -= 2;
        targetTr.localPosition = new Vector3(targetX, 2f, targetZ);

        foreach (var bodyPart in m_JdController.bodyPartsDict.Values)
        {
            bodyPart.Reset(bodyPart);
        }

        Invoke("InitGroundColor", 0.5f);
    }

    void InitGroundColor(){
        groundRd.material.color = groundBasicColor;
    }

    public override void Heuristic(float[] action)
    {
        for (int i = 0; i < 24; i++)
        {
            action[i] = Random.Range(-1f, 1f);
        }
    }
}
