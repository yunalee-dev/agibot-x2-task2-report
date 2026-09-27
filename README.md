# 작업 2 — 양팔 번갈아 X자 교차

[한국어 최종 HTML 보고서 열기](https://yunalee-dev.github.io/agibot-x2-task2-report/)

이 폴더에는 AgiBot X2 Ultra v1.3.0의 양팔을 번갈아 X자로 교차시키는 작업을 만들고, 전 경로의 관절 범위와 자기충돌을 검사한 결과가 들어 있습니다. A단계부터 I단계까지 구현과 수치 검증을 완료했으며, 최종 애니메이션도 함께 제공합니다.

## 핵심 결과

- 최종 상태: 운동학적 검증 통과
- 최종 반복: 5회
- 검사한 프레임: 4,251개
- 팔↔팔 최소거리: 0.070935214 m
- 관통: 0회
- 관절 한계 위반: 0회
- 자동 테스트: 31개 중 31개 통과
- 선택한 파라미터: `d140_v220`(교차 깊이 0.14 m, 높이 차 0.22 m)
- 공식 모델: `agibot_x2_urdf/X2_URDF-v1.3.0/x2_ultra.xml`
- 검사 설정: `configs/task2/phase_ab.json`

공식 MJCF·URDF·메시 파일은 수정하지 않았습니다. 기준 자세는 모델의 `qpos0`를 사용하고 속도는 0으로 두었으며, 설정에 포함된 양팔 14개 관절 위치만 변경했습니다.

## 단계별 완료 내용

| 단계 | 결과 | 수행 내용 |
|---|---|---|
| A | 통과 | 모델, 관절, 몸체, 충돌 형상, 메시와 좌표계 확인 |
| B | 통과 | 공통 기준 자세, 양팔 14개 관절 변경 제한, 부호 있는 거리의 의미 확인 |
| C | 통과·사용자 승인 | 양팔을 벌린 시작·복귀 자세 생성 |
| D | 통과·사용자 승인 | 왼팔이 위인 X자 교차 자세 생성 및 최근접 표면 관계 확인 |
| E | 통과·사용자 승인 | 오른팔이 위인 X자 교차 자세 생성 및 최근접 표면 관계 확인 |
| F | 통과·사용자 승인 | 2회 반복, 1,701개 프레임의 연속 궤적 검사 |
| G1 | 임시 보정 완료 | 충돌 거리 분포와 검토 대상 형상 쌍 산출 |
| G2 | 작업 범위 내 임시 정책 통과 | 안전·근접·충돌·회복 데이터를 용도별로 분리 |
| H | 통과 | 9개 파라미터 조합을 비교하고 `d140_v220` 선택 |
| I | 최종 운동학 검증 통과 | 5회 반복, 4,251개 프레임 최종 검사 |

## 동작 순서

한 동작 묶음은 다음 순서로 구성됩니다.

`펼친 자세 → 왼팔 위 교차 → 펼친 자세 → 오른팔 위 교차 → 펼친 자세`

최소 저크 보간을 사용해 각 자세 사이를 연결하고, 100 Hz로 모든 중간 프레임을 검사했습니다.

## 충돌 데이터와 ACT 데이터의 분리

충돌 상태가 ACT의 정답 행동으로 들어가지 않도록 데이터를 다음처럼 분리했습니다.

- 안전 상태: 관통이 없고 팔↔팔 거리가 작업 2 임시 기준 20 mm 이상
- 근접 상태: 관통은 없지만 팔↔팔 거리가 20 mm 미만
- 충돌 상태: 검사 범위의 부호 있는 거리 중 하나라도 0보다 작음
- 회복 시연: 근접 상태에서 승인된 안전 자세로 빠져나오는 검증된 궤적

분류 결과는 안전 977개, 근접 45개, 충돌 89개입니다. 충돌 89개는 판별기 학습이나 평가를 위한 부정 표본이며 ACT 행동 정답에는 사용하지 않습니다. 행동 학습에는 승인된 안전 시연과 전 경로를 다시 검사한 회복 시연만 사용합니다.

프로젝트 전체의 충돌 형상, 형상 쌍 집합과 양수 안전 임계값은 아직 `TBD-005`, `TBD-006`, `TBD-009` 상태입니다. 여기서 사용한 20 mm 기준은 작업 2 분석용 임시 값이며 공식 안전 기준이 아닙니다.

## 실행 방법

저장소 루트에서 PowerShell로 다음 명령을 실행합니다.

```powershell
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.audit_phase_ab
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_c
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_d
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_e
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_f
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_g1
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_g2
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_h
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.run_phase_i
& 'C:\Users\lyn04\anaconda3\python.exe' -m unittest discover -s tests -v
```

최종 궤적을 MuJoCo 뷰어에서 반복 재생하려면 다음 명령을 사용합니다.

```powershell
& 'C:\Users\lyn04\anaconda3\python.exe' -m scripts.task2.play_trajectory --trajectory results/task2/final_trajectory.csv --repeats 1 --loop
```

## 주요 결과 파일

### 최종 보고서와 시각 자료

- `task2_final_report.html`: 한국어 최종 보고서
- `task2_final.gif`: 최종 동작 애니메이션
- `open_front.png`, `open_side.png`: 펼친 자세의 정면·측면 이미지
- `phase_h_selected_left_over_right_front.png`, `phase_h_selected_left_over_right_side.png`: 왼팔 위 교차 자세
- `phase_h_selected_right_over_left_front.png`, `phase_h_selected_right_over_left_side.png`: 오른팔 위 교차 자세
- `g2_arm_arm_distance.png`: 한 동작의 팔 사이 최소거리 그래프
- `phase_h_status_heatmap.png`: H단계 조합별 검사 상태
- `phase_h_clearance_heatmap.png`: H단계 조합별 팔 사이 최소거리

### 최종 수치 데이터

- `final_trajectory.csv`: 최종 5회 반복 관절 궤적
- `final_distance_records.csv`: 최종 전 프레임 거리 기록
- `final_episode_summary.csv`: 반복별 요약
- `final_manifest.json`: 생성물과 검증 결과 목록
- `task2_field_contract.md`: 최종 데이터 필드 정의

### 데이터 분리 결과

- `g2_labeled_states.csv`: 안전·근접·충돌 라벨 상태
- `g2_negative_collision_states.csv`: 충돌 부정·평가 데이터
- `g2_safe_demonstration.csv`: 안전 시연
- `g2_recovery_demonstrations.csv`: 회복 시연
- `g2_recovery_manifest.csv`: 회복 시연 목록

### 파라미터 탐색 결과

- `phase_h_sweep.csv`: 9개 파라미터 조합의 전체 결과
- `phase_h_selected.json`: 최종 선택 근거와 수치
- `phase_h_report.md`: H단계 상세 보고서

## 실행 환경

- Python 3.13.5
- NumPy 2.1.3
- MuJoCo 3.14.0
- SciPy 1.15.3: 제한이 있는 비선형 최소제곱 역기구학에 사용
- Pillow 11.1.0: PNG와 GIF 저장 및 표시 문구 작성에 사용

이 작업을 위해 새 의존성을 설치하지 않았습니다.

## 해석할 때 주의할 점

이번 결과는 현재 MuJoCo 모델과 임시 검사 범위 안에서 관통이 발견되지 않았다는 뜻입니다. 실제 하드웨어의 안전을 인증한 결과는 아닙니다.

- `qpos`를 직접 입력하고 `mj_forward`를 수행한 운동학적 검사입니다.
- 모터 토크, 추종 오차, 균형, 관성, 케이블과 외피 변형은 검증하지 않았습니다.
- 현재 팔 충돌 형상은 주로 상세 시각 메시를 재사용합니다.
- 원본 모델에는 손목 요 회전축의 충돌 표면이 빠져 있습니다.
- 전체 최소거리 약 5.65 mm는 팔끼리가 아니라 어깨 회전부와 몸통 사이의 설계상 가까운 간격입니다.

물리적 안전거리를 주장하려면 공식 v1.3.0 모델을 그대로 보존한 채, 별도 버전의 분석용 MJCF에 팔 링크의 캡슐 또는 복합 볼록 대리 형상과 누락된 손목 표면을 추가하고 원본 메시와의 오차를 검증해야 합니다. 충돌 형상이나 형상 쌍 집합을 바꾸면 `docs/07_AGENT_RULES.md`에 따라 Silver Robotics 이후 계층을 다시 처리해야 합니다.
