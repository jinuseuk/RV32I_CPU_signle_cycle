# 🚀 RV32I Single-Cycle CPU Implementation

## 🎯 프로젝트 개요 (Project Overview)

본 프로젝트는 RISC-V (Reduced Instruction Set Computer - Five) 아키텍처의 기본 정수 명령어 세트인 RV32I를 지원하는 싱글 사이클(Single-Cycle) 프로세서를 SystemVerilog/Verilog로 구현한 것입니다.

싱글 사이클 아키텍처는 모든 명령어가 단 하나의 클럭 사이클 내에 실행되도록 설계되어, 마이크로프로세서의 기본 동작 원리(Fetch, Decode, Execute, Memory, Write-Back)를 명확하게 이해하고 구현하는 데 초점을 맞췄습니다.

  * 핵심 목표: RV32I 명령어 세트의 데이터패스(Datapath)와 제어 로직(Control Logic)을 하드웨어 기술 언어로 완벽하게 구현 및 검증.
  * 아키텍처: Harvard Architecture (명령어 메모리와 데이터 메모리 분리)
  * ISA 지원: RISC-V Base Integer Instruction Set (RV32I)

## ⚙️ 시스템 아키텍처 (System Architecture: Single-Cycle)

프로세서는 크게 데이터패스(Datapath)와 제어 장치(Control Unit)로 구성됩니다. 모든 구성 요소는 하나의 클럭 사이클 내에서 동작하며, 제어 장치가 명령어의 Opcode를 기반으로 모든 데이터패스 요소의 동작을 제어합니다.

### 1\. 명령어 지원 범위 (Supported Instructions)

| 타입 | 명령어 (예시) | 역할 |
| :--- | :--- | :--- |
| R-Type | ADD, SUB, XOR, OR, AND, SLL, SRL, SRA, SLT, SLTU | 레지스터 간 산술 및 논리 연산 |
| I-Type | ADDI, LW, LH, LB, JALR | 즉치값(Immediate)을 포함한 연산 및 메모리 Load, 점프 |
| S-Type | SW, SH, SB | 메모리 Store 연산 |
| B-Type | BEQ, BNE, BLT, BGE, BLTU, BGEU | 조건부 분기(Branch) |
| U-Type | LUI, AUIPC | 상위 20비트 즉치값 로드 |
| J-Type | JAL | 무조건 점프 및 링크 |

### 2\. 모듈 구성 (Module Breakdown)

RTL 코드는 RISC-V 프로세서의 표준 구성 요소를 기반으로 모듈화되어 있습니다.

| 모듈 파일 (추정) | 모듈 역할 (기능) | 상세 내용 |
| :--- | :--- | :--- |
| `rv32i_cpu_top.sv` | TOP (최상위 모듈 | 전체 모듈들을 통합하고 클럭 및 리셋 신호를 분배하여 CPU를 완성 |
| `PC.sv` | Program Counter | 다음에 실행할 명령어의 주소(PC)를 저장 및 업데이트 (PC + 4, Branch 주소 등) |
| `IMem.sv` | Instruction Memory | Hex 파일 형태의 프로그램 코드를 저장하고, PC 주소에 따라 명령어를 Fetch |
| `RegFile.sv` | Register File | 32개의 32비트 범용 레지스터(x0\~x31) 집합. 2개의 Read Port, 1개의 Write Port 제공 |
| `ALU.sv` | Arithmetic Logic Unit | 산술(ADD, SUB) 및 논리(AND, OR, XOR 등) 연산을 수행 |
| `DMem.sv` | Data Memory | 프로그램 실행 중 데이터를 저장하고 로드/스토어(Load/Store) 요청을 처리 |
| `Controller.sv` | Control Unit | 명령어의 Opcode 및 Funct3/7 필드를 디코딩하여, 데이터패스에 필요한 모든 제어 신호 (e.g., RegWrite, MemtoReg, ALUSrc, Branch, Jump, MemRead/Write)를 생성 |
| `Sign_Extend.sv` | Sign Extender | 12비트, 20비트 등의 즉치값(Immediate Value)을 32비트로 부호 확장 |

## 💻 구현 및 검증 (Implementation & Verification)

### 1\. SystemVerilog 구현 특징

  * Combinational vs. Sequential 각 모듈 내에서 조합 로직(`assign`, `always_comb`)과 순차 로직(`always_ff`)을 명확히 구분하여 설계했습니다.
  * Data Hazard / Control Hazard 미처리 Single-Cycle 설계의 특성상 파이프라인(Pipeline)이 없으므로, 데이터 및 제어 해저드(Hazard)를 고려하지 않아 구현의 복잡성을 낮췄습니다. (모든 명령어는 1 사이클 완료 보장)

### 2\. 시뮬레이션 환경

프로젝트의 기능을 검증하기 위해 별도의 테스트벤치(`rv32i_tb.sv` 추정)가 사용됩니다.

1.  Instruction Loading: RISC-V 어셈블리 코드를 컴파일하여 생성된 `.hex` 또는 `.dat` 파일을 `IMem`에 로드합니다.
2.  Testbench Run: 클럭을 인가하고 리셋을 해제하여 CPU 동작을 시작합니다.
3.  Verification: 시뮬레이션 후, GPR(General Purpose Registers) 값과 Data Memory의 최종 상태를 예상 결과와 비교하여 명령어 실행의 정확성을 검증합니다.


## 💡 학습 성과 및 가치 (Key Achievements)

  * 프로세서 구조 이해:Von Neumann/Harvard 아키텍처 및 싱글 사이클 디자인의 장단점을 직접 구현하며 체득.
  * ISA 디코딩 마스터: 복잡한 RISC-V 명령어 포맷(R, I, S, B, U, J-Type)을 분석하고, 이를 제어 신호로 변환하는 Control Unit 설계 역량 확보.
  * HDL 설계 능력: SystemVerilog를 사용하여 32비트 연산 및 데이터 경로를 포함하는 대규모 디지털 시스템을 모듈화하고 통합하는 능력 검증.
  * 디버깅 경험: PC 업데이트 오류, Branch/Jump 주소 계산 오류 등 CPU 설계 과정에서 발생하는 복잡한 타이밍 및 논리 오류를 디버깅하여 하드웨어 안정성 확보.
