# SystemVerilog / UVM Verification

SystemVerilog 클래스 기반 테스트벤치부터 UVM 1.2까지, 작은 DUT로 검증 환경을 단계별로 만들어 본 저장소입니다.
transaction, generator, driver, monitor, scoreboard를 직접 클래스로 나눠 본 다음, 같은 구조를 UVM의 sequence, driver, monitor, agent, scoreboard, subscriber로 옮겼고, 마지막에 기능 커버리지를 추가했습니다.
여기서 익힌 구조로 [RV32I Multi-Cycle CPU + UVM](https://github.com/gurwn031107-web/proj_rv32i_multi_cycle_cpu) 팀 프로젝트의 검증 환경을 만들었습니다.

- 기간: 2026.08.18 ~ 2026.09.05
- 도구: Synopsys VCS, Verdi, UVM 1.2

## 구성

| 폴더 | DUT | 내용 |
|---|---|---|
| [`01_adder_sv_tb`](01_adder_sv_tb) | 8-bit Adder | 클래스 테스트벤치: transaction, generator, driver, monitor, scoreboard를 mailbox와 event로 연결, virtual interface 사용 |
| [`02_register_sv_tb`](02_register_sv_tb) | 8-bit Register | 레지스터 랜덤 검증, `fork-join` / `join_any` / `join_none` 동작 차이 실습 |
| [`03_hello_uvm`](03_hello_uvm) | - | SystemVerilog 클래스, 상속, `super.new()` 기초 |
| [`04_sv_oop`](04_sv_oop) | - | OOP 예제 8개: 상속, static 멤버, virtual 함수와 다형성 |
| [`05_adder_uvm`](05_adder_uvm) | 8-bit Adder | 기본 UVM 구조(sequence → driver → monitor → scoreboard), 랜덤 100회 |
| [`06_counter_uvm`](06_counter_uvm) | 3-bit Counter | enable / reset 시퀀스를 나눠 tick 출력 검증 |
| [`07_ram_uvm`](07_ram_uvm) | 256 x 8 RAM | write / read 랜덤 검증, `uvm_subscriber`로 주소와 읽기 / 쓰기 기능 커버리지, 데이터 값 커버리지 버전 |
| [`08_fifo_sv_tb`](08_fifo_sv_tb) | 8-bit x 16 FIFO | 클래스 테스트벤치, full / empty 경계와 상태 전이 커버리지 버전 |

## 검증 환경 구조 (UVM)

```
test
└── env
    ├── agent
    │   ├── sequencer ← sequence (랜덤 transaction 생성)
    │   ├── driver    → virtual interface → DUT
    │   └── monitor   ← virtual interface ← DUT
    ├── scoreboard    ← monitor (기대값과 비교)
    └── coverage      ← monitor (uvm_subscriber, covergroup)
```

## 커버리지에서 본 것

- **RAM**: 쓰기 / 읽기 동작과 주소 0 ~ 255가 모두 접근됐는지 확인하고, 데이터 값 커버리지 버전에서는 쓴 값과 읽은 값 0 ~ 255가 모두 나왔는지 따로 확인
- **FIFO**: reset, push / pop 요청 조합(동시 요청 포함), 가득 찬 상태에서 push, 빈 상태에서 pop, full / empty가 바뀌는 순간까지 bin으로 나누고, 랜덤만으로는 잘 안 나오는 경계 상황은 FIFO를 끝까지 채우고 비우는 시나리오로 채움

## 실행

Makefile이 있는 폴더에서 VCS로 컴파일하고 시뮬레이션합니다.

```bash
make run      # 컴파일 (05는 make run 한 번에 실행까지)
make simv     # 시뮬레이션
make verdi    # 파형 확인
```

Makefile이 없는 01, 02, 04는 `rtl/`과 `tb/` 파일을 VCS나 Vivado 시뮬레이터에 함께 넣어 실행합니다.
