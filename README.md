# Implementarea unui Procesor RISC-V cu Pipeline pe 5 Etape

Proiect academic ce prezintă implementarea completă a unui procesor RISC-V cu pipeline pe 5 etape (IF, ID, EX, MEM, WB), realizat în cadrul cursului de Arhitectura Procesoarelor Moderne.

## Continut

### Lucrare Combinată

Fișierul [`Lucrare_Combinata_RISC_V.pdf`](Lucrare_Combinata_RISC_V.pdf) conține documentația completă a proiectului, împărțită în două părți:

- **Partea I** - Front-end-ul procesorului (etapele IF și ID)
- **Partea II** - Back-end-ul procesorului (etapele EX, MEM, WB, unitatea de forwarding și detectia hazardurilor)

### Implementare Verilog

#### `IF_ID/` - Front-end (Pipeline IF → ID)

Implementarea etapelor de pre-procesare:
- **IF (Instruction Fetch)** - Citirea instrucțiunii din memorie și incrementarea PC-ului
- **ID (Instruction Decode)** - Decodarea instrucțiunii, citirea registrilor și generarea valorilor imediate

Module principale:
| Modul | Descriere |
|-------|-----------|
| `IF.v` | Modulul principal IF |
| `ID.v` | Modulul principal ID |
| `PC.v` | Program Counter |
| `instruction_memory.v` | Memoria de instrucțiuni |
| `registers.v` | Bancul de registri (32x32-bit) |
| `imm_gen.v` | Generatorul de valori imediate |
| `if_id_pipe.v` | Registrul de pipeline IF/ID |
| `mux2_1.v`, `mux41.v` | Multiplexoare |
| `control_path.v` | Unitatea de control |
| `adder.v` | Sumator |

#### `RISC_V_EX_MEM_WB/` - Back-end (Pipeline EX → MEM → WB)

Implementarea etapelor de execuție și scriere:
- **EX (Execute)** - Execuția operațiilor ALU și calculul adresei de salt
- **MEM (Memory)** - Accesul la memoria de date
- **WB (Write Back)** - Scrierea rezultatelor în bancul de registri

Module principale:
| Modul | Descriere |
|-------|-----------|
| `EX.v` | Modulul principal EX |
| `ALU.v` | Unitatea ALU |
| `ALUcontrol.v` | Controlul ALU |
| `data_memory.v` | Memoria de date |
| `forwarding.v` | Unitatea de Forwarding (rezolvare hazarduri RAW) |
| `hazard_detection.v` | Unitatea de detectie a hazardurilor |
| `EX_MEM.v` | Registrul de pipeline EX/MEM |
| `MEM_WB.v` | Registrul de pipeline MEM/WB |
| `mux41.v` | Multiplexor 4:1 |
| `and_gate.poarta` | Poarta AND |

## Arhitectura Procesorului

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│    IF   │───▶│    ID   │───▶│    EX   │───▶│   MEM   │───▶│    WB   │
│         │    │         │    │         │    │         │    │         │
│ - PC    │    │ - Decode│    │ - ALU   │    │ - Data  │    │ - Write │
│ - IMEM  │    │ - Regs  │    │ - Forward│   │   Memory│    │   Back  │
│ - Adder │    │ - ImmGen│    │ - Branch│    │         │    │         │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
                  ▲                                  │
                  │         ┌─────────────┐          │
                  └─────────│   Hazard    │◀─────────┘
                            │  Detection  │
                            └─────────────┘
```

## Instrucțiuni Suportate

Procesorul suportă extensia **RV32I** (32-bit):

- **Aritmetico-logice**: `add`, `sub`, `and`, `or`, `xor`, `sll`, `srl`, `sra`, `slt`, `sltu`
- **Cu imediat**: `addi`, `andi`, `ori`, `xori`, `slli`, `srli`, `srai`, `slti`, `sltiu`
- **Memorie**: `lw`, `sw`
- **Branch**: `beq`, `bne`, `blt`, `bge`, `bltu`, `bgeu`

## Detecția și Rezolvarea Hazardurilor

- **Hazarduri RAW**: Rezolvate prin **forwarding** (bypass din etapele MEM/WB)
- **Load-Use Hazard**: Rezolvat prin **stalling** (inserare de bubble în pipeline)

## Rularea Simulării

1. Deschide proiectul în **Vivado**
2. Adaugă toate fișierele `.v` din cele două directoare
3. Rulează simularea folosind fișierul `test_riscv.v`
4. Verifică formele de undă pentru corectitudinea execuției

## Rezultate Simulare

După execuția codului assembly de test:
```
add  x2, x1, x0
addi x1, x1, 1
and  x3, x1, x2
ori  x4, x1, 1
sw   x4, 4(x5)
lw   x12, 8(x0)
beq  x18, x0, 5c
```

Valoarea finală a registrilor și a memoriei de date poate fi verificată în fereastra de simulație.

## Bibliografie

1. David A. Patterson, John L. Hennessy, *Computer Organization and Design RISC-V Edition*, 2018
2. Andrew Waterman, Krste Asanović, *The RISC-V Instruction Set Manual - Volume I: User-Level ISA*, 2017
