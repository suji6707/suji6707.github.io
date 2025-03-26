+++
title = 'Langraph Persistence'
date = 2025-03-26T22:50:44+09:00
draft = true
+++


"What’s really neat" 부분에서 언급된 핵심은 **LangGraph의 데이터베이스 기반 체크포인터 기능이 기본적인 세션 간 상태 저장(persistence)을 넘어, 체크포인트와 분기(branch point)를 활용하여 워크플로우의 특정 상태로 되돌아가거나 작업을 재실행(replay)할 수 있게 지원한다는 점**입니다. 이를 통해 사용자는 복잡한 작업 흐름에서도 유연하고 제어 가능한 방식으로 코드를 실행하고 관리할 수 있습니다.

### 주요 개념 설명

#### **기본 상태 저장(persistence)**:
일반적인 상태 저장 기능은 현재 진행 중인 워크플로우의 상태 정보를 데이터베이스에 저장하여, 프로세스가 종료된 후에도 이 정보를 유지할 수 있도록 합니다. 이는 주로 실패한 작업 복구나 세션 간 작업 이어서 수행 등에 활용됩니다.

#### **체크포인트(checkpoints)**:
체크포인트는 워크플로우의 특정 **중간 상태를 캡처**하여 저장하는 기능입니다. 이렇게 저장된 상태 정보는 이후에 복구할 수 있어, 특정 시점부터 워크플로우를 다시 실행하거나 수정 작업을 할 수 있습니다.

#### **분기(branch points)**:
분기는 워크플로우를 다른 경로 진행하거나 분기점에서 여러 가지 실행 상태를 생성할 수 있도록 처리하는 기능입니다. 이는 워크플로우가 복잡한 상태 관리나 선택 기반 작업을 수반하는 경우에 도움이 됩니다.

---

### 예시로 "undo 및 replay" 설명

#### 시나리오:
사용자가 AI 코드 생성 워크플로우를 실행 중이며, 여러 단계를 통해 작업을 수행합니다:
1. **`context_collector`:** 문맥 데이터를 수집.
2. **`task_planner`:** 계획을 수립.
3. **`task_executor`:** 실행하여 코드를 생성.
4. **`validator`:** 결과를 검증.

이 워크플로우는 PostgresSaver를 사용해 각 상태를 데이터베이스에 저장하며, 중간 상태와 작업 결과를 체크포인트로 기록합니다.

#### 예:
1. 처음 사용자 입력:
   ```python
   initial_state = {"user_input": "Write code for sorting algorithm"}
   result = graph.invoke(initial_state)
   print(result)
   ```
   - 이 호출에서 워크플로우가 완료되면, 전 단계의 상태(`context`, `plan`, `result` 등)가 DB에 저장됩니다.

2. 사용자가 결과에 만족하지 않고 실행 과정 중 특정 상태로 되돌아가서 수정해야 하는 상황:
   - 사용자는 `validator` 단계에서 실패했을 때, 워크플로우를 특정 지점으로 **되돌리기(undo)**:
     ```python
     checkpoints = checkpointer.list_checkpoints()
     checkpoint_id = checkpoints[0].id  # 첫번째 체크포인트 ID 가져오기
     graph.restore_state(checkpoint_id)  # 실시간 체크포인트로 되돌림
     
 # 수정된 입력 데이터를 전달하여 다시 실행
     new_state = {"user_input": "Improve the sorting algorithm"}
     updated_result = graph.invoke(new_state)
     print(updated_result)
     ```
     - 여기서 사용자는 `context_collector`, `task_planner` 단계의 결과를 유지하면서 다음 단계를 다시 실행하게 됩니다.

3. 사용자가 특정 조건에 따라 **다른 워크플로우 경로를 분기(branch)**:
   ```python
   graph.add_conditional_edges(
       "validator",
       should_revise,  # 조건 함수
       {
           True: "task_executor",  # 재실행 분기
           False: END # 종료 분기
       }
   )
   ```
   - 이분기를 기반으로, `validator` 단계에서 실패했을 경우, `task_executor`로 되돌아가 수정된 계획을 실행하며 새로운 결과를 생성합니다.

#### 결과:
- 체크포인트를 활용하여 오류가 있거나 불만족스러운 결과를 수정할 수 있습니다.
- 체크포인트에서 되돌아가거나 특정 경로 **재실행(replay)**하면서 워크플로우를 유연하게 제어할 수 있습니다.

---

### 추가 시나리오: Undo and Replay 활용

#### 예제 1. 작업 복구(Undo)
셰션이 중단되었다고 가정:
```python
# 이전 세션 체크포인트 목록 가져오기
checkpoints = checkpointer.list_checkpoints()
print(checkpoints)

# 가장 최신 체크포인트 선택 및 복구
latest_checkpoint_id = checkpoints[-1].id
restored_graph = checkpointer.load_checkpoint(latest_checkpoint_id)

# 복구된 상태에서 워크플로우 실행 재개
result = restored_graph.invoke()
print(result)
```

이 코드는 사용자의 이전 워크플로우 상태를 복구하여 중단된 작업을 이어서 수행합니다.

---

#### 예제 2. 워크플로우 변경(Replay + 수정):
사용자가 기존 결과를 변경하고 또 다른 사용자 입력으로 워크플로우를 재설정:
```python
# 기존 체크포인트로 복구
checkpoint_id = checkpoints[1].id
restored_graph = checkpointer.load_checkpoint(checkpoint_id)

# 수정된 입력으로 워크플로우 다시 실행
updated_input = {"user_input": "Optimize sorting algorithm to handle larger arrays"}
new_result = restored_graph.invoke(updated_input)

print(new_result)
```
이는 워크플로우를 특정 포인트로 되돌려, 새로운 입력과 함께 작업을 다시 실행하거나 수정된 결과를 생성할 수 있습니다.

---

### 요약:
LangGraph의 체크포인터(`PostgresSaver`)는 기본적인 세션 간 상태 저장만 제공하는 것이 아니라:
1. 특정 워크플로우 상태로 **되돌아가기(undo)** 기능 제공.
2. 중간 상태를 재실행(replay)하여 수정 및 확장을 간단하게 처리.
3. 조건부 논리와 분기(branch)를 통해 복잡한 워크플로우 유도 가능.

이를 통해 사용자는 단순 반복 실행에 그치지 않고, **워크플로우를 유연하고 조절 가능한 방식으로 관리**할 수 있습니다.