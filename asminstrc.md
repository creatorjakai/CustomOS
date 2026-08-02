# Assembly-instructiereferentie — x86-64 en ARM64/AArch64

> Bestandsnaam: `asminstrc.md`  
> Taal: Nederlands  
> Scope: CPU-instructies voor x86-64 en AArch64/ARM64. Linux-syscalls, kernel-API's en OS-specifieke ABI-functies zijn niet opgenomen.

## Belangrijke afbakening

Deze referentie beschrijft de **architecturale instructies en instructiefamilies** die je in 64-bit assembly tegenkomt. Een letterlijk overzicht van iedere mogelijke machinecode-encoding zou vele duizenden regels beslaan. Eén mnemonic kan tientallen of honderden operand-, vectorlengte-, mask-, broadcast- en datatypevormen hebben. Daarom worden vormen met dezelfde werking als één familie beschreven.

- **x86-64:** algemene integerinstructies, x87, MMX, SSE-familie, AVX/AVX2, FMA, BMI, AES/SHA, AVX-512, AMX en belangrijke AMD/Intel-uitbreidingen.
- **ARM64:** A64-basisinstructies, Advanced SIMD/NEON, floating point, atomics, crypto, pointer authentication, SVE/SVE2 en SME-families.
- Privileged en virtualisatie-instructies zijn wel als CPU-instructies genoemd, maar duidelijk gemarkeerd. Ze zijn niet zonder meer vanuit een normaal gebruikersprogramma uitvoerbaar.
- Assemblers kennen vaak **aliassen**. Bijvoorbeeld ARM64 `MOV`, `CMP`, `TST`, `NEG` en `CSET` kunnen aliassen van andere encodings zijn.

## Notatie

| Symbool | Betekenis |
|---|---|
| `dst` | Bestemming |
| `src` | Bron |
| `r/m` | Register of geheugenoperand |
| `imm` | Directe constante |
| `rel` | Relatieve sprongafstand |
| `cc` / `cond` | Conditiecode |
| `Xn` | ARM64 64-bit algemeen register |
| `Wn` | ARM64 32-bit algemeen register |
| `Vn` | ARM64 SIMD/FP-register |
| `Zn` | ARM SVE-vectorregister |
| `Pn` | ARM SVE-predicaatregister |
| `ZF/SF/CF/OF` | x86 zero/sign/carry/overflowflags |
| `N/Z/C/V` | ARM negative/zero/carry/overflowflags |

---

# Deel I — x86-64

## 1. Gegevens verplaatsen en adressen

| Instructie | Gebruik | Werking | Flags / extensie |
|---|---|---|---|
| `MOV` | `mov dst, src` | Kopieert integerdata tussen registers en geheugen of laadt een immediate. Geen geheugen-naar-geheugen-vorm. | Geen |
| `MOVABS` | `movabs r64, imm64` | Assemblernaam voor een volledige 64-bit immediate of absoluut adres. | Geen |
| `MOVZX` | `movzx r32/r64, r/m8/r/m16` | Laadt en vult de hogere bits met nullen. | Geen |
| `MOVSX` | `movsx r16/r32/r64, r/m8/r/m16` | Laadt en vult de hogere bits met het tekenbit. | Geen |
| `MOVSXD` | `movsxd r64, r/m32` | Sign-extend van 32 naar 64 bits. | Geen |
| `LEA` | `lea r64, [adres]` | Berekent een effectief adres zonder geheugen te lezen. Ook bruikbaar voor eenvoudige rekenexpressies. | Geen |
| `XCHG` | `xchg a, b` | Verwisselt twee waarden. Met geheugenoperand atomisch. | Geen |
| `BSWAP` | `bswap r32/r64` | Keert de bytevolgorde om. | 486+ |
| `XLAT` / `XLATB` | `xlatb` | Laadt byte uit tabel op adres `RBX + AL` naar `AL`. | Geen |
| `PUSH` | `push r/m64/imm` | Verlaagt `RSP` en schrijft een stackwaarde. | Geen |
| `POP` | `pop r/m64` | Leest stackwaarde en verhoogt `RSP`. | Geen |
| `PUSHFQ` | `pushfq` | Zet `RFLAGS` op de stack. | Geen |
| `POPFQ` | `popfq` | Herstelt toegestane delen van `RFLAGS`. | Sommige bits privileged |
| `LAHF` | `lahf` | Kopieert statusflags naar `AH`. | Geen |
| `SAHF` | `sahf` | Kopieert `AH` naar statusflags. | Geen |
| `LDS`, `LES` | legacy | Laadt pointer plus segmentregister. Niet beschikbaar als normale 64-bit instructie. | Legacy |
| `LFS`, `LGS`, `LSS` | `lfs/lgs/lss reg, mem` | Laadt offset en segmentselector. | Systeemgericht |

## 2. Integerrekenen

| Instructie | Gebruik | Werking | Flags |
|---|---|---|---|
| `ADD` | `add dst, src` | Optellen. | `OF,SF,ZF,AF,CF,PF` |
| `ADC` | `adc dst, src` | Optellen inclusief carry. | Rekenvlaggen |
| `ADCX` | `adcx dst, src` | Optellen met alleen `CF` als carryketen. | `CF`; ADX |
| `ADOX` | `adox dst, src` | Optellen met alleen `OF` als carryketen. | `OF`; ADX |
| `SUB` | `sub dst, src` | Aftrekken. | Rekenvlaggen |
| `SBB` | `sbb dst, src` | Aftrekken inclusief borrow via `CF`. | Rekenvlaggen |
| `INC` | `inc dst` | Verhoogt met één. | Alles behalve `CF` |
| `DEC` | `dec dst` | Verlaagt met één. | Alles behalve `CF` |
| `NEG` | `neg dst` | Tweecomplement-negatie: `0-dst`. | Rekenvlaggen |
| `CMP` | `cmp a, b` | Berekent conceptueel `a-b`, bewaart alleen flags. | Rekenvlaggen |
| `MUL` | `mul r/m` | Unsigned vermenigvuldiging met impliciete accumulator; dubbelbreed resultaat. | `CF,OF` relevant |
| `IMUL` | `imul ...` | Signed vermenigvuldiging; één-, twee- en drie-operandvormen. | `CF,OF` bij overflow |
| `MULX` | `mulx hi, lo, src` | Unsigned dubbelbreed vermenigvuldigen zonder flags te wijzigen. | BMI2 |
| `DIV` | `div r/m` | Unsigned deling van dubbelbrede accumulator. Quotient en rest impliciet. | Flags undefined |
| `IDIV` | `idiv r/m` | Signed deling. | Flags undefined |
| `CBW` | `cbw` | Sign-extend `AL` naar `AX`. | Geen |
| `CWDE` | `cwde` | Sign-extend `AX` naar `EAX`. | Geen |
| `CDQE` | `cdqe` | Sign-extend `EAX` naar `RAX`. | Geen |
| `CWD` | `cwd` | Sign-extend `AX` naar `DX:AX`. | Geen |
| `CDQ` | `cdq` | Sign-extend `EAX` naar `EDX:EAX`. | Geen |
| `CQO` | `cqo` | Sign-extend `RAX` naar `RDX:RAX`. | Geen |
| `AAA`, `AAS`, `AAM`, `AAD`, `DAA`, `DAS` | legacy decimal | Oude BCD/ASCII-correctie-instructies; niet beschikbaar in 64-bit mode. | Legacy |

## 3. Logica, bits en shifts

| Instructie | Gebruik | Werking | Flags / extensie |
|---|---|---|---|
| `AND` | `and dst, src` | Bitwise AND. | `SF,ZF,PF`; `CF=OF=0` |
| `OR` | `or dst, src` | Bitwise OR. | `SF,ZF,PF`; `CF=OF=0` |
| `XOR` | `xor dst, src` | Bitwise XOR. `xor reg,reg` maakt efficiënt nul. | `SF,ZF,PF`; `CF=OF=0` |
| `NOT` | `not dst` | Keert ieder bit om. | Geen |
| `TEST` | `test a, b` | AND voor flags, zonder resultaat op te slaan. | Zoals `AND` |
| `SHL` / `SAL` | `shl dst, count` | Logisch links schuiven. Beide namen zijn dezelfde instructie. | Shiftflags |
| `SHR` | `shr dst, count` | Logisch rechts schuiven met nullen. | Shiftflags |
| `SAR` | `sar dst, count` | Rekenkundig rechts schuiven met tekenbit. | Shiftflags |
| `SHLX` | `shlx dst, src, count` | Links schuiven zonder flags. | BMI2 |
| `SHRX` | `shrx dst, src, count` | Logisch rechts zonder flags. | BMI2 |
| `SARX` | `sarx dst, src, count` | Rekenkundig rechts zonder flags. | BMI2 |
| `ROL` | `rol dst, count` | Bits links roteren. | `CF`, soms `OF` |
| `ROR` | `ror dst, count` | Bits rechts roteren. | `CF`, soms `OF` |
| `RCL` | `rcl dst, count` | Links roteren door carry. | `CF`, soms `OF` |
| `RCR` | `rcr dst, count` | Rechts roteren door carry. | `CF`, soms `OF` |
| `RORX` | `rorx dst, src, imm` | Rechts roteren zonder flags. | BMI2 |
| `SHLD` | `shld dst, src, count` | Dubbelprecisie links shift: bits komen uit tweede operand. | Shiftflags |
| `SHRD` | `shrd dst, src, count` | Dubbelprecisie rechts shift. | Shiftflags |
| `BT` | `bt base, bit` | Kopieert geselecteerd bit naar `CF`. | `CF` |
| `BTS` | `bts base, bit` | Test en zet bit. | `CF` |
| `BTR` | `btr base, bit` | Test en wis bit. | `CF` |
| `BTC` | `btc base, bit` | Test en keer bit om. | `CF` |
| `BSF` | `bsf dst, src` | Index van laagste ingestelde bit. | `ZF` |
| `BSR` | `bsr dst, src` | Index van hoogste ingestelde bit. | `ZF` |
| `TZCNT` | `tzcnt dst, src` | Telt trailing zeros. | BMI1 |
| `LZCNT` | `lzcnt dst, src` | Telt leading zeros. | ABM/LZCNT |
| `POPCNT` | `popcnt dst, src` | Telt ingestelde bits. | POPCNT |
| `ANDN` | `andn dst, a, b` | `(~a) & b`. | BMI1 |
| `BEXTR` | `bextr dst, src, control` | Extraheert a aaneengesloten bitveld. | BMI1 |
| `BLSI` | `blsi dst, src` | Isoleert laagste ingestelde bit. | BMI1 |
| `BLSMSK` | `blsmsk dst, src` | Masker tot en met laagste ingestelde bit. | BMI1 |
| `BLSR` | `blsr dst, src` | Wist laagste ingestelde bit. | BMI1 |
| `BZHI` | `bzhi dst, src, index` | Maakt alle bits boven index nul. | BMI2 |
| `PDEP` | `pdep dst, src, mask` | Verspreidt bits volgens masker. | BMI2 |
| `PEXT` | `pext dst, src, mask` | Verzamelt bits volgens masker. | BMI2 |
| `CRC32` | `crc32 dst, src` | CRC-32C-stap. | SSE4.2 |

## 4. Control flow en voorwaarden

| Instructie | Werking | Conditie / opmerkingen |
|---|---|---|
| `JMP` | Onvoorwaardelijke directe of indirecte sprong. | Geen |
| `CALL` | Zet returnadres op stack en springt. | Direct of indirect |
| `RET` / `RETN` | Haalt returnadres van stack. | Optioneel stackadjustment |
| `RETF` | Far return met segmentwisseling. | Systeem/legacy |
| `JE` / `JZ` | Springt bij gelijk/nul. | `ZF=1` |
| `JNE` / `JNZ` | Springt bij ongelijk/niet nul. | `ZF=0` |
| `JA` / `JNBE` | Unsigned groter. | `CF=0 && ZF=0` |
| `JAE` / `JNB` / `JNC` | Unsigned groter of gelijk. | `CF=0` |
| `JB` / `JNAE` / `JC` | Unsigned kleiner. | `CF=1` |
| `JBE` / `JNA` | Unsigned kleiner of gelijk. | `CF=1 || ZF=1` |
| `JG` / `JNLE` | Signed groter. | `ZF=0 && SF=OF` |
| `JGE` / `JNL` | Signed groter of gelijk. | `SF=OF` |
| `JL` / `JNGE` | Signed kleiner. | `SF!=OF` |
| `JLE` / `JNG` | Signed kleiner of gelijk. | `ZF=1 || SF!=OF` |
| `JO` / `JNO` | Springt bij wel/geen overflow. | `OF=1/0` |
| `JS` / `JNS` | Springt bij negatief/niet negatief. | `SF=1/0` |
| `JP` / `JPE` | Springt bij even parity. | `PF=1` |
| `JNP` / `JPO` | Springt bij oneven parity. | `PF=0` |
| `JCXZ` / `JECXZ` / `JRCXZ` | Springt als `CX/ECX/RCX` nul is. | Registergrootte afhankelijk |
| `LOOP` | Decrementeert teller en springt wanneer niet nul. | `RCX`-familie |
| `LOOPE` / `LOOPZ` | Loop zolang teller niet nul en `ZF=1`. |  |
| `LOOPNE` / `LOOPNZ` | Loop zolang teller niet nul en `ZF=0`. |  |
| `CMOVcc` | Kopieert operand wanneer conditie waar is. | `CMOVE`, `CMOVNE`, `CMOVA`, enz. |
| `SETcc` | Schrijft byte 0 of 1 volgens conditie. | `SETE`, `SETNE`, `SETA`, enz. |
| `ENTER` | Maakt procedure-stackframe. | Vaak langzamer dan losse instructies |
| `LEAVE` | Herstelt stackframe via `RBP`. | Equivalent aan `mov rsp,rbp; pop rbp` |
| `IRETQ` | Keert terug uit interrupt/exception. | Privileged/contextafhankelijk |

### Alle x86-conditiecodes

`A, AE, B, BE, C, E, G, GE, L, LE, NA, NAE, NB, NBE, NC, NE, NG, NGE, NL, NLE, NO, NP, NS, NZ, O, P, PE, PO, S, Z`.

Deze suffixen bestaan waar ondersteund voor `Jcc`, `CMOVcc` en `SETcc`.

## 5. String- en herhaalinstructies

| Instructie | Werking |
|---|---|
| `MOVSB`, `MOVSW`, `MOVSD`, `MOVSQ` | Kopieert element van `[RSI]` naar `[RDI]`; pointers volgen `DF`. |
| `CMPSB`, `CMPSW`, `CMPSD`, `CMPSQ` | Vergelijkt elementen bij `[RSI]` en `[RDI]`. `CMPSD` kan ook SSE betekenen afhankelijk van operands. |
| `SCASB`, `SCASW`, `SCASD`, `SCASQ` | Vergelijkt accumulator met element bij `[RDI]`. |
| `LODSB`, `LODSW`, `LODSD`, `LODSQ` | Laadt element bij `[RSI]` naar accumulator. |
| `STOSB`, `STOSW`, `STOSD`, `STOSQ` | Slaat accumulator op bij `[RDI]`. |
| `INSB`, `INSW`, `INSD` | Leest I/O-poort naar geheugen. Privileged/I/O-permission nodig. |
| `OUTSB`, `OUTSW`, `OUTSD` | Schrijft geheugen naar I/O-poort. Privileged/I/O-permission nodig. |
| `REP` | Herhaalt stringinstructie `RCX` keer. |
| `REPE` / `REPZ` | Herhaalt zolang gelijk/nul. |
| `REPNE` / `REPNZ` | Herhaalt zolang ongelijk/niet nul. |
| `CLD` | Maakt direction flag nul: pointers lopen omhoog. |
| `STD` | Zet direction flag: pointers lopen omlaag. |

## 6. Flags en algemene processorbesturing

| Instructie | Werking | Toegang |
|---|---|---|
| `CLC` | Wist carryflag. | User |
| `STC` | Zet carryflag. | User |
| `CMC` | Keert carryflag om. | User |
| `CLD` | Wist direction flag. | User |
| `STD` | Zet direction flag. | User |
| `CLI` | Maskeert interrupts. | Privileged |
| `STI` | Activeert interrupts. | Privileged |
| `NOP` | Geen architecturaal effect. | User |
| `PAUSE` | Hint voor spin-waitloops. | User |
| `HLT` | Stopt core tot event/interrupt. | Privileged |
| `WAIT` / `FWAIT` | Wacht op pending x87-exceptions. | User |
| `UD2` | Genereert gegarandeerd invalid-opcode exception. | User |
| `INT3` | Genereert breakpoint-exception. | User/debugger |
| `INT imm8` | Software-interrupt. | Poortrechten/DPL bepalen toegang |
| `INTO` | Interrupt bij overflow; niet in 64-bit mode. | Legacy |
| `BOUND` | Boundscheck; niet in 64-bit mode. | Legacy |

## 7. Synchronisatie, geheugenordening en atomics

| Instructie/prefix | Werking | Extensie |
|---|---|---|
| `LOCK` | Maakt ondersteunde read-modify-write-instructie atomisch ten opzichte van andere cores. | Basis |
| `CMPXCHG` | Vergelijkt accumulator met bestemming; vervangt bij gelijk. | Basis |
| `CMPXCHG8B` | Atomische compare-exchange van 64 bits. | Pentium |
| `CMPXCHG16B` | Atomische compare-exchange van 128 bits. | CX16 |
| `XADD` | Verwisselt en telt atomisch op met `LOCK`. | 486 |
| `XCHG` | Met geheugenoperand impliciet atomisch. | Basis |
| `LFENCE` | Load fence en bepaalde uitvoeringsserialisatie. | SSE2 |
| `SFENCE` | Store fence. | SSE |
| `MFENCE` | Volledige geheugenfence. | SSE2 |
| `CLFLUSH` | Schrijft cachelijn terug en invalideert die. | CLFSH |
| `CLFLUSHOPT` | Geoptimaliseerde cachelineflush. | CLFLUSHOPT |
| `CLWB` | Schrijft cachelijn terug zonder verplichte invalidatie. | CLWB |
| `CLDEMOTE` | Hint om cachelijn naar lager cacheniveau te verplaatsen. | CLDEMOTE |
| `MOVNTI` | Non-temporal integerstore. | SSE2 |
| `MOVNTDQ`, `MOVNTDQA`, `MOVNTPD`, `MOVNTPS`, `MOVNTQ` | Non-temporal SIMD-load/storevormen. | SSE/AVX-familie |
| `MONITOR` / `MWAIT` | Monitor geheugenadres en slaap tot write/event. | Meestal privileged/configuratieafhankelijk |
| `MONITORX` / `MWAITX` | AMD-variant met timeout/hints. | AMD |
| `UMONITOR`, `UMWAIT`, `TPAUSE` | User-mode wait/monitor-instructies. | WAITPKG |

## 8. I/O-, descriptor- en systeeminstructies

> Deze zijn CPU-instructies, geen Linux-syscalls. Veel zijn privileged en horen bij OS-, hypervisor- of firmwarecode.

| Instructie | Werking |
|---|---|
| `IN`, `OUT` | Leest/schrijft x86-I/O-poort. |
| `CPUID` | Leest CPU-identificatie en featurebits. |
| `RDTSC` | Leest timestamp counter. |
| `RDTSCP` | Leest timestamp counter plus auxiliary ID en gedeeltelijke ordering. |
| `RDPMC` | Leest performance-monitoringcounter indien toegestaan. |
| `RDMSR`, `WRMSR` | Leest/schrijft model-specific register. Privileged. |
| `RDPID` | Leest processor-ID uit `IA32_TSC_AUX`. |
| `LGDT`, `SGDT` | Laadt/leest global descriptor table-register. |
| `LIDT`, `SIDT` | Laadt/leest interrupt descriptor table-register. |
| `LLDT`, `SLDT` | Laadt/leest local descriptor table-selector. |
| `LTR`, `STR` | Laadt/leest task register. |
| `LMSW`, `SMSW` | Laadt/leest machine status word. |
| `CLTS` | Wist task-switched-bit in `CR0`. |
| `MOV CRn` | Leest/schrijft controlregisters. Privileged. |
| `MOV DRn` | Leest/schrijft debugregisters. |
| `INVLPG` | Invalideert TLB-entry voor pagina. |
| `INVPCID` | Invalideert vertalingen volgens PCID-type. |
| `WBINVD` | Write-back en invalidate caches. |
| `INVD` | Invalideert caches zonder write-back. |
| `SWAPGS` | Wisselt GS-base met kernel-GS-base. |
| `RDFSBASE`, `RDGSBASE` | Leest FS/GS-base wanneer enabled. |
| `WRFSBASE`, `WRGSBASE` | Schrijft FS/GS-base wanneer enabled. |
| `SYSENTER`, `SYSEXIT` | Snelle ringovergang, vooral 32-bit ABI. |
| `SYSCALL`, `SYSRET` | Architecturale snelle privilegeovergang. OS-specifiek gebruik; geen syscalllijst opgenomen. |
| `RSM` | Keert terug uit System Management Mode. |
| `SERIALIZE` | Volledig serialiserende instructie. |
| `ENCLS`, `ENCLU`, `ENCLV` | Intel SGX leaf-dispatchers. |
| `GETSEC` | Intel SMX/Safer Mode Extensions. |
| `SKINIT` | AMD secure init. |

## 9. Random, cryptografie en checksum

| Instructie | Werking | Extensie |
|---|---|---|
| `RDRAND` | Levert hardware-randomwaarde; `CF` geeft succes. | RDRAND |
| `RDSEED` | Levert seed-grade randomwaarde; `CF` geeft succes. | RDSEED |
| `AESENC`, `AESENCLAST` | AES encryptieronde / laatste ronde. | AES-NI |
| `AESDEC`, `AESDECLAST` | AES decryptieronde / laatste ronde. | AES-NI |
| `AESIMC` | Inverse MixColumns voor round key. | AES-NI |
| `AESKEYGENASSIST` | Assisteert AES-key schedule. | AES-NI |
| `PCLMULQDQ` / `VPCLMULQDQ` | Carry-less vermenigvuldiging van 64-bit polynomen. | PCLMULQDQ/VPCLMULQDQ |
| `SHA1MSG1`, `SHA1MSG2`, `SHA1NEXTE`, `SHA1RNDS4` | SHA-1-acceleratie. | Intel SHA |
| `SHA256MSG1`, `SHA256MSG2`, `SHA256RNDS2` | SHA-256-acceleratie. | Intel SHA |
| `GF2P8AFFINEQB`, `GF2P8AFFINEINVQB`, `GF2P8MULB` | GF(2^8)-bewerkingen. | GFNI |
| `CRC32` | CRC-32C-update. | SSE4.2 |

## 10. x87 floating-point

De x87 gebruikt een registerstack `ST(0)` tot `ST(7)` en 80-bit extended precision intern.

| Familie / instructies | Werking |
|---|---|
| `FLD`, `FILD`, `FBLD` | Laadt floating-point, integer of packed BCD op x87-stack. |
| `FST`, `FSTP`, `FIST`, `FISTP`, `FISTTP`, `FBSTP` | Slaat op, meestal met optionele pop/conversie. |
| `FXCH` | Wisselt `ST(0)` met ander stackregister. |
| `FADD`, `FADDP`, `FIADD` | Floating-point optellen. |
| `FSUB`, `FSUBP`, `FSUBR`, `FSUBRP`, `FISUB`, `FISUBR` | Aftrekken, eventueel omgekeerde operandvolgorde. |
| `FMUL`, `FMULP`, `FIMUL` | Vermenigvuldigen. |
| `FDIV`, `FDIVP`, `FDIVR`, `FDIVRP`, `FIDIV`, `FIDIVR` | Delen. |
| `FSQRT` | Vierkantswortel. |
| `FSCALE` | Schaal met macht van twee. |
| `FPREM`, `FPREM1` | Gedeeltelijke restberekening. |
| `FRNDINT` | Rondt naar integer volgens control word. |
| `FABS`, `FCHS` | Absolute waarde / teken omkeren. |
| `FCOM`, `FCOMP`, `FCOMPP`, `FUCOM`, `FUCOMP`, `FUCOMPP` | Vergelijkt via x87-statuswoord. |
| `FCOMI`, `FCOMIP`, `FUCOMI`, `FUCOMIP` | Vergelijkt en schrijft integerflags. |
| `FTST`, `FXAM` | Test tegen nul / classificeert waarde. |
| `FSIN`, `FCOS`, `FSINCOS`, `FPTAN`, `FPATAN` | Trigonometrische bewerkingen. |
| `F2XM1`, `FYL2X`, `FYL2XP1` | Exponentiële/logaritmische hulpfuncties. |
| `FLD1`, `FLDZ`, `FLDPI`, `FLDL2E`, `FLDL2T`, `FLDLG2`, `FLDLN2` | Laadt ingebouwde constanten. |
| `FINIT` / `FNINIT` | Initialiseert x87-state. |
| `FCLEX` / `FNCLEX` | Wist x87-exceptions. |
| `FLDCW`, `FSTCW` / `FNSTCW` | Laadt/leest control word. |
| `FSTSW` / `FNSTSW` | Leest statusword. |
| `FSAVE` / `FNSAVE`, `FRSTOR` | Slaat/herstelt oude x87-state. |
| `FXSAVE`, `FXRSTOR`, `FXSAVE64`, `FXRSTOR64` | Slaat/herstelt x87+SSE-state. |
| `XSAVE`, `XSAVEC`, `XSAVEOPT`, `XSAVES` | Slaat uitgebreide CPU-state op. |
| `XRSTOR`, `XRSTORS` | Herstelt uitgebreide CPU-state. |
| `XGETBV`, `XSETBV` | Leest/schrijft extended control register. |
| `FINCSTP`, `FDECSTP`, `FFREE`, `FFREEP` | Beheert x87-stacktop/tags. |
| `FNOP` | x87 no-op. |

## 11. MMX en klassieke packed-integer SIMD

MMX gebruikt `MM0`–`MM7`, die fysieke state delen met x87. `EMMS` of `FEMMS` maakt de x87-tags weer bruikbaar.

| Familie | Mnemonics / werking |
|---|---|
| State | `EMMS`, AMD `FEMMS` — beëindigt MMX-gebruik. |
| Move | `MOVD`, `MOVQ` — verplaatst 32/64-bit data. |
| Packed add | `PADDB`, `PADDW`, `PADDD`, `PADDQ`; saturerend `PADDSB`, `PADDSW`, `PADDUSB`, `PADDUSW`. |
| Packed subtract | `PSUBB`, `PSUBW`, `PSUBD`, `PSUBQ`; saturerend `PSUBSB`, `PSUBSW`, `PSUBUSB`, `PSUBUSW`. |
| Multiply | `PMULLW`, `PMULHW`, `PMULHUW`, `PMADDWD`. |
| Compare | `PCMPEQB/W/D`, `PCMPGTB/W/D`. |
| Logic | `PAND`, `PANDN`, `POR`, `PXOR`. |
| Shift | `PSLLW/D/Q`, `PSRLW/D/Q`, `PSRAW/D`. |
| Pack | `PACKSSWB`, `PACKSSDW`, `PACKUSWB`. |
| Unpack | `PUNPCKHBW/HWD/HDQ`, `PUNPCKLBW/LWD/LDQ`. |

## 12. SSE, SSE2, SSE3, SSSE3 en SSE4

### Datamoves en shuffles

| Instructiefamilie | Werking |
|---|---|
| `MOVAPS`, `MOVUPS` | Aligned/unaligned packed single-precision move. |
| `MOVAPD`, `MOVUPD` | Aligned/unaligned packed double-precision move. |
| `MOVSS`, `MOVSD` | Scalar single/double move. |
| `MOVD`, `MOVQ`, `MOVDQ2Q`, `MOVQ2DQ` | Integer/SIMD/MMX transfers. |
| `MOVDQA`, `MOVDQU` | Aligned/unaligned packed integer move. |
| `MOVHLPS`, `MOVLHPS`, `MOVHPS`, `MOVLPS`, `MOVHPD`, `MOVLPD` | Verplaatst hoge/lage delen. |
| `MOVMSKPS`, `MOVMSKPD`, `PMOVMSKB` | Extraheert signbits naar integerregister. |
| `SHUFPS`, `SHUFPD`, `PSHUFD`, `PSHUFHW`, `PSHUFLW`, `PSHUFB` | Herschikt lanes/bytes. |
| `UNPCKHPS`, `UNPCKLPS`, `UNPCKHPD`, `UNPCKLPD` | Interleavet hoge/lage lanes. |
| `PUNPCKH*`, `PUNPCKL*` | Integerinterleave voor byte/word/dword/qword. |
| `PALIGNR` | Concatenateert en verschuift bytes. |
| `PBLENDW`, `BLENDPS`, `BLENDPD`, `PBLENDVB`, `BLENDVPS`, `BLENDVPD` | Selecteert lanes volgens immediate of mask. |
| `INSERTPS`, `EXTRACTPS`, `PINSRB/W/D/Q`, `PEXTRB/W/D/Q` | Voegt lane in of extraheert lane. |

### Floating-pointrekenen

Voor de meeste onderstaande instructies bestaan `PS` (packed single), `PD` (packed double), `SS` (scalar single) en `SD` (scalar double)-vormen.

| Familie | Mnemonics | Werking |
|---|---|---|
| Add | `ADDPS`, `ADDPD`, `ADDSS`, `ADDSD` | Optellen. |
| Sub | `SUBPS`, `SUBPD`, `SUBSS`, `SUBSD` | Aftrekken. |
| Multiply | `MULPS`, `MULPD`, `MULSS`, `MULSD` | Vermenigvuldigen. |
| Divide | `DIVPS`, `DIVPD`, `DIVSS`, `DIVSD` | Delen. |
| Square root | `SQRTPS`, `SQRTPD`, `SQRTSS`, `SQRTSD` | Vierkantswortel. |
| Approx reciprocal | `RCPPS`, `RCPSS` | Benaderde reciproke waarde. |
| Approx reciprocal sqrt | `RSQRTPS`, `RSQRTSS` | Benaderde reciproke wortel. |
| Min/max | `MINPS/PD/SS/SD`, `MAXPS/PD/SS/SD` | Minimum/maximum volgens SSE-regels. |
| Horizontal | `HADDPS`, `HADDPD`, `HSUBPS`, `HSUBPD` | Horizontale add/sub. |
| Add-sub | `ADDSUBPS`, `ADDSUBPD` | Afwisselend aftrekken en optellen. |
| Dot product | `DPPS`, `DPPD` | Dotproduct met immediate-masker. |
| Round | `ROUNDPS`, `ROUNDPD`, `ROUNDSS`, `ROUNDSD` | Rondt volgens gekozen modus. |

### Vergelijken en conversies

| Familie | Werking |
|---|---|
| `CMPPS`, `CMPPD`, `CMPSS`, `CMPSD` | Vergelijkt met immediate predicate en levert maskers. |
| `COMISS`, `UCOMISS`, `COMISD`, `UCOMISD` | Scalar compare naar integerflags. |
| `CVTDQ2PS`, `CVTDQ2PD`, `CVTPS2DQ`, `CVTTPS2DQ`, `CVTPD2DQ`, `CVTTPD2DQ` | Packed integer/floatconversies. |
| `CVTPS2PD`, `CVTPD2PS` | Single/double-conversie. |
| `CVTSI2SS`, `CVTSI2SD`, `CVTSS2SI`, `CVTTSS2SI`, `CVTSD2SI`, `CVTTSD2SI` | Scalar integer/floatconversies. |
| `CVTSS2SD`, `CVTSD2SS` | Scalar precisieconversie. |
| `PMOVSX*`, `PMOVZX*` | Sign-/zero-extend packed elementen naar bredere lanes. |

### Packed integerrekenen

| Familie | Mnemonics |
|---|---|
| Add/sub | `PADDB/W/D/Q`, `PSUBB/W/D/Q`; signed/unsigned saturerende vormen. |
| Multiply | `PMULLW`, `PMULLD`, `PMULHW`, `PMULHUW`, `PMULDQ`, `PMULUDQ`, `PMADDWD`, `PMADDUBSW`. |
| Min/max | `PMIN*`, `PMAX*` voor signed/unsigned B/W/D afhankelijk van extensie. |
| Average | `PAVGB`, `PAVGW`. |
| Absolute | `PABSB`, `PABSW`, `PABSD`. |
| Sign | `PSIGNB`, `PSIGNW`, `PSIGND`. |
| SAD | `PSADBW`, `MPSADBW`. |
| Compare | `PCMPEQB/W/D/Q`, `PCMPGTB/W/D/Q`. |
| Logic | `PAND`, `PANDN`, `POR`, `PXOR`. |
| Shift | `PSLLW/D/Q`, `PSRLW/D/Q`, `PSRAW/D`, `PSLLDQ`, `PSRLDQ`. |
| Pack/unpack | `PACKSSWB`, `PACKSSDW`, `PACKUSDW`, `PACKUSWB`, `PUNPCK*`. |
| String compare | `PCMPESTRI/M`, `PCMPISTRI/M`. |
| Test | `PTEST`. |

### SSE-state en cache

| Instructie | Werking |
|---|---|
| `LDMXCSR` / `STMXCSR` | Laadt/leest SIMD floating-point control/status. |
| `MASKMOVDQU` | Gemaskeerde byte-store. |
| `PREFETCHNTA`, `PREFETCHT0`, `PREFETCHT1`, `PREFETCHT2` | Cache-prefetchhints. |
| `PREFETCHW`, `PREFETCHWT1` | Prefetch voor write of specifieke cachehint. |

## 13. AVX en AVX2

AVX voegt meestal een `V` toe aan SSE-mnemonics, gebruikt niet-destructieve drie-operandvormen en ondersteunt 256-bit `YMM`-registers. AVX2 breidt vrijwel alle packed-integerbewerkingen naar 256 bits uit.

| Familie | Voorbeelden / werking |
|---|---|
| AVX floating point | `VADDPS/PD/SS/SD`, `VSUB*`, `VMUL*`, `VDIV*`, `VSQRT*`, `VMIN*`, `VMAX*`, `VCMP*`. |
| AVX moves | `VMOVAPS/UPS/APD/UPD/DQA/DQU/SS/SD`, non-temporal varianten. |
| AVX shuffle/blend | `VSHUFPS/PD`, `VPERMILPS/PD`, `VPERM2F128`, `VBLENDPS/PD/V`, `VINSERTF128`, `VEXTRACTF128`. |
| AVX conversion | `VCVT*`-vormen van SSE-conversies. |
| AVX2 integer | `VPADD*`, `VPSUB*`, `VPMUL*`, `VPMIN*`, `VPMAX*`, `VPCMPEQ*`, `VPCMPGT*`, `VPACK*`, `VPUNPCK*`. |
| AVX2 variable shifts | `VPSLLVD/Q`, `VPSRLVD/Q`, `VPSRAVD`. |
| AVX2 permutes | `VPERMD`, `VPERMPS`, `VPERMQ`, `VPERMPD`, `VPERM2I128`. |
| Gather | `VGATHERDPS/DPD/QPS/QPD`, `VPGATHERDD/DQ/QD/QQ`. |
| Mask moves | `VMASKMOVPS`, `VMASKMOVPD`, `VPMASKMOVD`, `VPMASKMOVQ`. |
| Zero upper | `VZEROUPPER`, `VZEROALL` — voorkomt AVX/SSE-overgangsproblemen of wist vectorstate. |
| Broadcast | `VBROADCASTSS`, `VBROADCASTSD`, `VBROADCASTF128`, `VPBROADCASTB/W/D/Q`. |

## 14. FMA, F16C en andere vectoruitbreidingen

| Familie | Mnemonics / werking |
|---|---|
| FMA3 | `VFMADD132/213/231PS/PD/SS/SD`, `VFMSUB*`, `VFNMADD*`, `VFNMSUB*`, `VFMADDSUB*`, `VFMSUBADD*`. Eén afronding voor `a*b±c`. |
| FMA4 | AMD vier-operandvormen zoals `VFMADDPD/PS/SD/SS`, `VFMSUB*`, `VFNMADD*`, `VFNMSUB*`. |
| F16C | `VCVTPH2PS`, `VCVTPS2PH` — converteert half precision en single precision. |
| XOP | AMD-families zoals `VPCMOV`, `VPPERM`, `VPMAC*`, `VPROT*`, `VPSHA*`, `VPSHL*`, `VFRCZ*`, `VPHADD*`, `VPHSUB*`. |
| TBM | AMD `BEXTR`-achtige en trailing-bitinstructies: `BLCFILL`, `BLCI`, `BLCIC`, `BLCMSK`, `BLCS`, `BLSFILL`, `BLSIC`, `T1MSKC`, `TZMSK`. |

## 15. AVX-512

AVX-512 gebruikt `ZMM0`–`ZMM31`, maskregisters `K0`–`K7`, optionele zero-masking `{z}`, broadcasts en vaak embedded rounding/SAE.

| Categorie | Belangrijkste families |
|---|---|
| Foundation arithmetic | `VADD*`, `VSUB*`, `VMUL*`, `VDIV*`, `VSQRT*`, `VMIN*`, `VMAX*`, `VSCALEF*`, `VRNDSCALE*`, `VREDUCE*`, `VRANGE*`, `VGETEXP*`, `VGETMANT*`, `VFIXUPIMM*`. |
| FMA | Alle `VFMADD*`, `VFMSUB*`, `VFNMADD*`, `VFNMSUB*`, addsub/subadd-vormen met masking. |
| Compare | `VCMP*`, `VPCMPB/UB/W/UW/D/UD/Q/UQ`, `VPCMPEQ*`, `VPCMPGT*`; resultaat vaak in `K`-masker. |
| Mask registers | `KADDB/W/D/Q`, `KAND*`, `KANDN*`, `KMOV*`, `KNOT*`, `KOR*`, `KORTEST*`, `KSHIFTL*`, `KSHIFTR*`, `KTEST*`, `KUNPCK*`, `KXNOR*`, `KXOR*`. |
| Moves | `VMOVDQA32/64`, `VMOVDQU8/16/32/64`, `VMOVAPS/UPD/...` met masking. |
| Compress/expand | `VCOMPRESSPS/PD`, `VPCOMPRESSD/Q`, `VEXPANDPS/PD`, `VPEXPANDD/Q`. |
| Permute | `VPERMB/W/D/Q/PS/PD`, `VPERMI2*`, `VPERMT2*`, `VSHUFF32X4`, `VSHUFF64X2`, `VSHUFI32X4`, `VSHUFI64X2`. |
| Insert/extract | `VINSERTF32X4/F64X2/F32X8/F64X4`, `VINSERTI*`, `VEXTRACTF*`, `VEXTRACTI*`. |
| Broadcast | `VBROADCASTF32X2/X4/X8`, `VBROADCASTF64X2/X4`, `VBROADCASTI32X2/X4/X8`, `VBROADCASTI64X2/X4`, `VPBROADCAST*`. |
| Conversion | `VCVT*`-families tussen signed/unsigned integers, FP16/BF16/FP32/FP64, truncerende `VCVTT*`-vormen. |
| Integer arithmetic | `VPADD*`, `VPSUB*`, saturerende vormen, `VPMULL*`, `VPMULH*`, `VPMADD*`, `VPMULTISHIFTQB`, `VPDPBUSD/S`, `VPDPWSSD/S`. |
| Bit manipulation | `VPTERNLOGD/Q`, `VPLZCNTD/Q`, `VPOPCNTB/W/D/Q`, `VPSHLD*`, `VPSHRD*`, `VPSHUFBITQMB`, `VPCONFLICTD/Q`. |
| Gather/scatter | `VGATHER*`, `VPGATHER*`, `VSCATTER*`, `VPSCATTER*`. |
| Crypto | `VAESENC/DEC/ENCLAST/DECLAST`, `VPCLMULQDQ`, `VGF2P8*`, `VSHA512*`, `VSM3*`, `VSM4*` wanneer betreffende extensie aanwezig is. |
| Neural/BF16 | `VDPBF16PS`, `VCVTNEPS2BF16`, `VCVTNE2PS2BF16`. |
| FP16 | `VADDPH/SH`, `VSUBPH/SH`, `VMULPH/SH`, `VDIVPH/SH`, `VSQRTPH/SH`, FMA- en conversiefamilies. |
| Reciprocal | `VRCP14PS/PD/SS/SD`, `VRSQRT14*`, `VRCP28*`, `VRSQRT28*`. |
| 4FMAPS/4VNNIW | `V4FMADDPS`, `V4FNMADDPS`, `VP4DPWSSD`, `VP4DPWSSDS`. |

## 16. AMX — Advanced Matrix Extensions

| Instructie | Werking |
|---|---|
| `LDTILECFG` | Laadt tileconfiguratie. |
| `STTILECFG` | Slaat tileconfiguratie op. |
| `TILELOADD`, `TILELOADDT1` | Laadt 2D-tile uit geheugen. |
| `TILESTORED` | Slaat tile op. |
| `TILERELEASE` | Geeft tile-state vrij. |
| `TILEZERO` | Maakt tile nul. |
| `TDPBSSD`, `TDPBSUD`, `TDPBUSD`, `TDPBUUD` | INT8 dotproducts naar 32-bit accumulator. |
| `TDPBF16PS` | BF16-dotproduct naar FP32. |
| `TDPFP16PS` | FP16-dotproduct naar FP32. |
| `TCMMIMFP16PS`, `TCMMRLFP16PS` | Complexe FP16-matrix multiply-accumulate. |

## 17. Virtualisatie en beveiligde uitvoering

| Familie | Instructies | Doel |
|---|---|---|
| Intel VMX | `VMXON`, `VMXOFF`, `VMCLEAR`, `VMPTRLD`, `VMPTRST`, `VMREAD`, `VMWRITE`, `VMLAUNCH`, `VMRESUME`, `VMCALL`, `INVEPT`, `INVVPID`, `VMFUNC` | Hardwarevirtualisatie. Privileged. |
| AMD SVM | `VMRUN`, `VMMCALL`, `VMLOAD`, `VMSAVE`, `STGI`, `CLGI`, `INVLPGA`, `SKINIT` | AMD-hardwarevirtualisatie. Privileged. |
| CET | `ENDBR32`, `ENDBR64`, `RDSSPD`, `RDSSPQ`, `INCSSPD`, `INCSSPQ`, `SAVEPREVSSP`, `RSTORSSP`, `WRSSD`, `WRSSQ`, `WRUSSD`, `WRUSSQ`, `SETSSBSY`, `CLRSSBSY` | Control-flow enforcement/shadow stack. |
| MPX | `BNDMK`, `BNDCL`, `BNDCU`, `BNDCN`, `BNDMOV`, `BNDLDX`, `BNDSTX` | Deprecated memory bounds extension. |
| PKU | `RDPKRU`, `WRPKRU` | Leest/schrijft protection-key rights. |
| SEV-SNP | `PVALIDATE`, `RMPADJUST`, `RMPUPDATE`, `PSMASH` en gerelateerde AMD-instructies | Beveiligde VM-geheugenstatus; privileged. |

---

# Deel II — ARM64 / AArch64

## 18. Registers en basisregels

- `X0`–`X30`: 64-bit algemene registers.
- `W0`–`W30`: onderste 32 bits; schrijven naar `Wn` maakt de bovenste 32 bits van `Xn` nul.
- `SP`: stackpointer.
- `XZR` / `WZR`: zeroregister; lezen geeft nul, schrijven wordt weggegooid.
- `X30`: link register (`LR`) bij normale functieaanroepen.
- `V0`–`V31`: 128-bit Advanced SIMD/floating-pointregisters, met views `B/H/S/D/Q`.
- `Z0`–`Z31`, `P0`–`P15`, `FFR`: SVE-registers.
- `ZA`, `ZT0`: SME-matrix/tile-state wanneer aanwezig.

## 19. Integerrekenen

| Instructie/familie | Werking | Flags |
|---|---|---|
| `ADD` | Optellen, immediate of shifted/extended register. | Geen |
| `ADDS` | Optellen en `NZCV` zetten. | `N,Z,C,V` |
| `ADC` | Optellen inclusief carry. | Geen |
| `ADCS` | Optellen inclusief carry en flags zetten. | `NZCV` |
| `SUB` | Aftrekken. | Geen |
| `SUBS` | Aftrekken en flags zetten. | `NZCV` |
| `SBC` | Aftrekken met inverted-borrow/carry. | Geen |
| `SBCS` | Aftrekken met carry en flags. | `NZCV` |
| `NEG` | Alias voor `SUB dst, XZR, src`. | Geen |
| `NEGS` | Alias voor `SUBS dst, XZR, src`. | `NZCV` |
| `NGC` / `NGCS` | Negate with carry; alias van `SBC/SBCS` met zero-register. | Optioneel |
| `CMP` | Alias van `SUBS ...` met bestemming `XZR/WZR`. | `NZCV` |
| `CMN` | Alias van `ADDS ...` met bestemming `XZR/WZR`. | `NZCV` |
| `MUL` | Lage helft van vermenigvuldiging; alias van `MADD` met nuladdend. | Geen |
| `MNEG` | Negatief product; alias van `MSUB` met nul. | Geen |
| `MADD` | `a*b+c`. | Geen |
| `MSUB` | `c-a*b`. | Geen |
| `SMULL`, `UMULL` | Signed/unsigned 32×32 naar 64 bits. | Geen |
| `SMADDL`, `UMADDL` | Lange multiply-add. | Geen |
| `SMSUBL`, `UMSUBL` | Lange multiply-subtract. | Geen |
| `SMULH`, `UMULH` | Hoge 64 bits van 64×64-product. | Geen |
| `SDIV`, `UDIV` | Signed/unsigned integerdeling; delen door nul levert architecturaal nul. | Geen |

## 20. Logische bewerkingen en bitvelden

| Instructie | Werking |
|---|---|
| `AND` | Bitwise AND. |
| `ANDS` | AND en flags zetten. |
| `ORR` | Bitwise OR. `MOV reg,reg` is vaak alias hiervan. |
| `ORN` | OR met geïnverteerde tweede operand. |
| `EOR` | XOR. |
| `EON` | XOR met geïnverteerde tweede operand. |
| `BIC` | Bit clear: `a & ~b`. |
| `BICS` | Bit clear en flags. |
| `MVN` | Bitwise NOT; alias van `ORN` met zero-register. |
| `TST` | AND voor flags; alias van `ANDS` naar zero-register. |
| `LSL` | Logisch links schuiven; immediatevorm vaak alias van `UBFM`. |
| `LSR` | Logisch rechts schuiven; alias van `UBFM`. |
| `ASR` | Rekenkundig rechts schuiven; alias van `SBFM`. |
| `ROR` | Rechts roteren; immediatevorm alias van `EXTR`. |
| `LSLV`, `LSRV`, `ASRV`, `RORV` | Shift/rotate met variabele registerafstand. |
| `EXTR` | Extraheert bits uit concatenatie van twee registers. |
| `BFM` | Algemene bitfield move. |
| `SBFM` | Signed bitfield move met sign extension. |
| `UBFM` | Unsigned bitfield move/zero extension. |
| `BFI`, `BFXIL` | Bitfield insert / lage bitfield insert; aliassen van `BFM`. |
| `SBFX`, `UBFX` | Signed/unsigned bitfield extract; aliassen. |
| `SBFIZ`, `UBFIZ` | Bitfield insert in zero met sign/zero-semantiek. |
| `SXTB`, `SXTH`, `SXTW` | Sign-extend byte/halfword/word. |
| `UXTB`, `UXTH` | Zero-extend byte/halfword. `W`-registerwrites doen al 32→64 zero-extension. |
| `BFC` | Wist bitveld; alias van `BFM`. |
| `CLZ` | Telt leading zeros. |
| `CLS` | Telt leading signbits. |
| `RBIT` | Keert alle bits om. |
| `REV`, `REV16`, `REV32`, `REV64` | Keert bytevolgorde per gekozen elementgrootte om. |

## 21. Conditionele instructies

| Instructie | Werking |
|---|---|
| `CSEL` | Selecteert één van twee registers volgens conditie. |
| `CSINC` | Selecteert eerste operand, anders tweede plus één. |
| `CSINV` | Selecteert eerste operand, anders bitwise inverse tweede. |
| `CSNEG` | Selecteert eerste operand, anders negatieve tweede. |
| `CSET` | Schrijft 1 wanneer conditie waar is, anders 0; alias van `CSINC`. |
| `CSETM` | Schrijft alle bits 1 wanneer conditie waar is; alias van `CSINV`. |
| `CINC` | Conditioneel increment; alias van `CSINC`. |
| `CINV` | Conditioneel inverteren; alias van `CSINV`. |
| `CNEG` | Conditioneel negatief; alias van `CSNEG`. |
| `CCMP` | Vergelijkt conditioneel; anders worden immediate flags gebruikt. |
| `CCMN` | Conditionele compare-negative/add. |

### ARM-conditiecodes

| Code | Betekenis | Voorwaarde |
|---|---|---|
| `EQ` | gelijk | `Z=1` |
| `NE` | ongelijk | `Z=0` |
| `CS` / `HS` | carry / unsigned ≥ | `C=1` |
| `CC` / `LO` | geen carry / unsigned < | `C=0` |
| `MI` | negatief | `N=1` |
| `PL` | positief of nul | `N=0` |
| `VS` | overflow | `V=1` |
| `VC` | geen overflow | `V=0` |
| `HI` | unsigned > | `C=1 && Z=0` |
| `LS` | unsigned ≤ | `C=0 || Z=1` |
| `GE` | signed ≥ | `N=V` |
| `LT` | signed < | `N!=V` |
| `GT` | signed > | `Z=0 && N=V` |
| `LE` | signed ≤ | `Z=1 || N!=V` |
| `AL` | altijd | onvoorwaardelijk |
| `NV` | architecturaal gereserveerd/vaak always-achtig afhankelijk van encoding | Vermijd als algemene conditie |

## 22. Branches, calls en returns

| Instructie | Werking |
|---|---|
| `B label` | PC-relative onvoorwaardelijke branch. |
| `B.cond label` | Branch volgens `NZCV`, bijvoorbeeld `B.EQ`. |
| `BL label` | Branch with link; schrijft returnadres naar `X30`. |
| `BR Xn` | Indirecte branch naar register. |
| `BLR Xn` | Indirecte call; returnadres naar `X30`. |
| `RET Xn` | Return naar register, standaard `X30`. |
| `CBZ`, `CBNZ` | Branch wanneer register nul/niet nul is, zonder flags te wijzigen. |
| `TBZ`, `TBNZ` | Test één bit en branch bij nul/niet nul. |
| `BRK #imm` | Breakpoint-exception. |
| `HLT #imm` | Halting debug/exception-instructie; gebruik EL-/debugafhankelijk. |
| `UDF #imm` | Undefined-instruction encoding. |
| `ERET` | Return uit exception naar lagere/geselecteerde exception level. Privileged. |
| `ERETAA`, `ERETAB` | Exception return met pointer authentication. |
| `BRAA`, `BRAB`, `BRAAZ`, `BRABZ` | Geauthenticeerde indirecte branch. |
| `BLRAA`, `BLRAB`, `BLRAAZ`, `BLRABZ` | Geauthenticeerde indirecte call. |
| `RETAA`, `RETAB` | Return met authentication. |

## 23. Constanten en adressen maken

| Instructie | Werking |
|---|---|
| `MOVZ` | Schrijft 16-bit immediate in gekozen halfword en maakt rest nul. |
| `MOVN` | Schrijft bitwise inverse van immediate-halfwordconstructie. |
| `MOVK` | Vervangt één 16-bit halfword en behoudt overige bits. |
| `MOV` | Assembleralias voor geschikte `ORR`, `MOVZ`, `MOVN` of andere vorm. |
| `ADR` | Maakt PC-relative adres binnen ongeveer ±1 MiB. |
| `ADRP` | Maakt PC-relative 4-KiB-pageadres binnen groot bereik. Vaak gevolgd door `ADD` of `LDR`. |

## 24. Loads en stores

| Instructie/familie | Werking |
|---|---|
| `LDR` | Laadt register uit geheugen; integer, SIMD of literalvorm. |
| `STR` | Slaat register op in geheugen. |
| `LDRB`, `LDRH` | Laadt byte/halfword met zero-extension. |
| `STRB`, `STRH` | Slaat lage byte/halfword op. |
| `LDRSB`, `LDRSH`, `LDRSW` | Laadt signed byte/halfword/word met sign-extension. |
| `LDP` | Laadt registerpaar; ondersteunt offset, pre-index en post-index. |
| `STP` | Slaat registerpaar op. Veelgebruikt voor stackframes. |
| `LDNP`, `STNP` | Non-temporal pair load/store-hint. |
| `LDUR`, `STUR` | Unscaled signed-offset load/store. |
| `LDURB/H/SB/SH/SW`, `STURB/H` | Unscaled varianten per datatype. |
| `LDTR`, `STTR` en typed varianten | Unprivileged translation-regime load/store. |
| `PRFM` | Prefetch geheugen met hint. |
| `PRFUM` | Prefetch met unscaled offset. |
| `LDRAA`, `LDRAB` | Laadt pointer en authenticeert hem. |
| `LD64B`, `ST64B`, `ST64BV`, `ST64BV0` | 64-byte accelerator/atomic transferfamilie wanneer FEAT_LS64 aanwezig is. |
| `CPY*`, `SET*` | MOPS memory copy/set families met prologue/main/epiloguevormen wanneer FEAT_MOPS aanwezig is. |

### ARM64-adresseringsvormen

- Unsigned scaled offset: `[Xn, #imm]`
- Signed unscaled offset: `[Xn, #imm]`
- Register offset: `[Xn, Xm, extend/shift]`
- Pre-index: `[Xn, #imm]!`
- Post-index: `[Xn], #imm`
- Literal load: PC-relative adres in instructie

## 25. Atomics en exclusieve toegang

| Instructie/familie | Werking |
|---|---|
| `LDXR`, `LDXRB`, `LDXRH` | Exclusieve load. Zet exclusieve monitor. |
| `LDAXR`, `LDAXRB`, `LDAXRH` | Exclusieve load met acquire-semantiek. |
| `STXR`, `STXRB`, `STXRH` | Exclusieve store; statusregister meldt succes/mislukking. |
| `STLXR`, `STLXRB`, `STLXRH` | Exclusieve store met release-semantiek. |
| `LDXP`, `LDAXP` | Exclusieve load van registerpaar. |
| `STXP`, `STLXP` | Exclusieve store van registerpaar. |
| `CLREX` | Wist lokale exclusieve monitor. |
| `LDAR`, `LDARB`, `LDARH` | Acquire-load. |
| `STLR`, `STLRB`, `STLRH` | Release-store. |
| `LDAPR`, `LDAPRB`, `LDAPRH` | RCpc acquire-load. |
| `CAS`, `CASA`, `CASL`, `CASAL` | Compare-and-swap met relaxed/acquire/release/acq_rel semantiek. |
| `CASB/H`, `CASP` en orderingvarianten | Byte, halfword en pair compare-and-swap. |
| `SWP`, `SWPA`, `SWPL`, `SWPAL` | Atomic swap. Ook `B/H`-vormen. |
| `LDADD*` | Atomic fetch-add, met `A/L/AL` en `B/H`-vormen. |
| `LDCLR*` | Atomic fetch-and-clear bits. |
| `LDEOR*` | Atomic fetch-XOR. |
| `LDSET*` | Atomic fetch-OR/set bits. |
| `LDSMAX*`, `LDSMIN*` | Atomic signed max/min. |
| `LDUMAX*`, `LDUMIN*` | Atomic unsigned max/min. |
| `STADD*`, `STCLR*`, `STEOR*`, `STSET*`, `STSMAX*`, `STSMIN*`, `STUMAX*`, `STUMIN*` | Store-only aliassen wanneer oude waarde niet nodig is. |

## 26. Barrières, hints en events

| Instructie | Werking |
|---|---|
| `DMB option` | Data memory barrier; ordent expliciete geheugenaccesses. |
| `DSB option` | Data synchronization barrier; wacht tot relevante effecten voltooid zijn. |
| `ISB` | Instruction synchronization barrier; volgende instructies worden opnieuw gefetcht onder nieuwe context. |
| `SB` | Speculation barrier. |
| `SSBB`, `PSSBB` | Speculative store bypass barriers. |
| `CSDB` | Consumption of speculative data barrier. |
| `ESB` | Error synchronization barrier. |
| `PSB CSYNC` | Profiling synchronization barrier. |
| `TSB CSYNC` | Trace synchronization barrier. |
| `NOP` | Geen architecturaal effect. |
| `YIELD` | Scheduling/spinloop-hint. |
| `WFE` | Wacht op event. |
| `WFI` | Wacht op interrupt. Gewone uitvoering kan beperkt zijn per EL. |
| `SEV` | Stuurt event naar processing elements. |
| `SEVL` | Zet alleen lokale eventregisterconditie. |
| `HINT #imm` | Algemene hintencoding; diverse features definiëren specifieke hints. |

## 27. Floating-point scalar

| Familie | Mnemonics / werking |
|---|---|
| Move | `FMOV` — verplaatst FP-waarden, immediates of bits tussen FP en integerregisters. |
| Add/sub | `FADD`, `FSUB`. |
| Multiply/divide | `FMUL`, `FDIV`. |
| Negated multiply | `FNMUL`. |
| Fused multiply-add | `FMADD`, `FMSUB`, `FNMADD`, `FNMSUB`. |
| Square root | `FSQRT`. |
| Abs/neg | `FABS`, `FNEG`. |
| Min/max | `FMIN`, `FMAX`, `FMINNM`, `FMAXNM`; `NM` behandelt NaN anders. |
| Compare | `FCMP`, `FCMPE`; zet `NZCV`. |
| Conditional compare | `FCCMP`, `FCCMPE`. |
| Conditional select | `FCSEL`. |
| Round integral | `FRINTA`, `FRINTI`, `FRINTM`, `FRINTN`, `FRINTP`, `FRINTX`, `FRINTZ`, plus nieuwere `FRINT32X/Z`, `FRINT64X/Z`. |
| Convert FP precision | `FCVT` tussen half/single/double waar ondersteund. |
| FP→signed int | `FCVTAS`, `FCVTMS`, `FCVTNS`, `FCVTPS`, `FCVTZS`; afrondmodus in mnemonic. |
| FP→unsigned int | `FCVTAU`, `FCVTMU`, `FCVTNU`, `FCVTPU`, `FCVTZU`. |
| Int→FP | `SCVTF`, `UCVTF`. |
| Fixed-point convert | Dezelfde convertfamilies met fractional-bits immediate. |
| JavaScript convert | `FJCVTZS` — JavaScript-compatibele double→signed integerconversie. |

## 28. Advanced SIMD / NEON

Veel NEON-instructies bestaan voor meerdere laneformaten: `8B`, `16B`, `4H`, `8H`, `2S`, `4S`, `1D`, `2D`, plus scalarvormen.

### Integer en logica

| Familie | Instructies / werking |
|---|---|
| Add/sub | `ADD`, `SUB`; widening `SADDL/UADDL`, `SSUBL/USUBL`; high-half `SADDL2`, enz. |
| Add wide | `SADDW`, `UADDW`, `SSUBW`, `USUBW` en `...2`. |
| Halving | `SHADD`, `UHADD`, `SHSUB`, `UHSUB`, `SRHADD`, `URHADD`. |
| Saturating | `SQADD`, `UQADD`, `SQSUB`, `UQSUB`, `SUQADD`, `USQADD`. |
| Pairwise | `ADDP`, `SADALP`, `UADALP`, `SADDLP`, `UADDLP`. |
| Reduction | `ADDV`, `SADDLV`, `UADDLV`, `SMAXV`, `UMAXV`, `SMINV`, `UMINV`. |
| Multiply | `MUL`, `MLA`, `MLS`, widening `SMULL/UMULL`, `SMLAL/UMLAL`, `SMLSL/UMLSL`, high-half `...2`. |
| High multiply | `SQDMULH`, `SQRDMULH`, `SQDMULL`, `SQDMLAL`, `SQDMLSL`. |
| Dot product | `SDOT`, `UDOT`, `SUDOT`, `USDOT` afhankelijk van feature. |
| Matrix multiply | `SMMLA`, `UMMLA`, `USMMLA`, `BFMMLA`, `FMMLA` afhankelijk van feature. |
| Abs/difference | `ABS`, `NEG`, `SABD`, `UABD`, `SABDL`, `UABDL`, `SABA`, `UABA`, `SABAL`, `UABAL`. |
| Min/max | `SMIN`, `SMAX`, `UMIN`, `UMAX`, pairwise `SMINP`, enz. |
| Compare | `CMEQ`, `CMGE`, `CMGT`, `CMHI`, `CMHS`, `CMLE`, `CMLT`, `CMTST`. |
| Bitwise | `AND`, `ORR`, `ORN`, `EOR`, `BIC`, `BIT`, `BIF`, `BSL`, `NOT`. |
| Count/reverse | `CNT`, `CLZ`, `CLS`, `RBIT`, `REV16`, `REV32`, `REV64`. |
| Shift | `SHL`, `SSHL`, `USHL`, `SRSHL`, `URSHL`, immediate `SSHR/USHR`, `SRSHR/URSHR`, `SLI`, `SRI`. |
| Narrow shifts | `SHRN`, `RSHRN`, `SQSHRN`, `SQRSHRN`, `UQSHRN`, `UQRSHRN`, `SQSHRUN`, `SQRSHRUN`, plus `...2`. |
| Saturating left | `SQSHL`, `UQSHL`, `SQSHLU`; rounded registervormen. |

### Permutes, table lookups en moves

| Familie | Instructies / werking |
|---|---|
| Extract | `EXT` — concateneert twee vectoren en extraheert bytevenster. |
| Table lookup | `TBL`, `TBX`. |
| Zip/unzip | `ZIP1`, `ZIP2`, `UZP1`, `UZP2`. |
| Transpose | `TRN1`, `TRN2`. |
| Insert/move | `INS`, `DUP`, `SMOV`, `UMOV`. |
| Widen/narrow | `XTN`, `XTN2`, `SQXTN`, `SQXTN2`, `UQXTN`, `UQXTN2`, `SQXTUN`, `SQXTUN2`. |

### Floating-point SIMD

| Familie | Instructies / werking |
|---|---|
| Arithmetic | `FADD`, `FSUB`, `FMUL`, `FDIV`, `FMLA`, `FMLS`, `FNMUL`, `FMLAL`, `FMLSL` en high-halfvormen. |
| Pairwise/reduction | `FADDP`, `FMAXP`, `FMINP`, `FMAXNMP`, `FMINNMP`, `FMAXV`, `FMINV`, `FMAXNMV`, `FMINNMV`. |
| Abs/difference | `FABS`, `FNEG`, `FABD`. |
| Compare | `FCMEQ`, `FCMGE`, `FCMGT`, `FCMLE`, `FCMLT`, `FACGE`, `FACGT`. |
| Min/max | `FMIN`, `FMAX`, `FMINNM`, `FMAXNM`. |
| Reciprocal estimate/step | `FRECPE`, `FRECPS`, `FRSQRTE`, `FRSQRTS`, `FRECPX`. |
| Sqrt | `FSQRT`. |
| Round | `FRINTA/I/M/N/P/X/Z` en 32/64-bit varianten. |
| Convert | `FCVT*`, `SCVTF`, `UCVTF`, `FCVTL`, `FCVTN`, `FCVTXN` en `...2`. |
| Complex | `FCADD`, `FCMLA` wanneer complex-number extension aanwezig is. |

## 29. ARM cryptografie

| Instructiefamilie | Werking |
|---|---|
| AES | `AESE`, `AESD`, `AESMC`, `AESIMC`. |
| SHA-1 | `SHA1C`, `SHA1M`, `SHA1P`, `SHA1H`, `SHA1SU0`, `SHA1SU1`. |
| SHA-256 | `SHA256H`, `SHA256H2`, `SHA256SU0`, `SHA256SU1`. |
| SHA-512 | `SHA512H`, `SHA512H2`, `SHA512SU0`, `SHA512SU1`. |
| SHA-3 | `EOR3`, `RAX1`, `XAR`, `BCAX`. |
| SM3 | `SM3SS1`, `SM3TT1A`, `SM3TT1B`, `SM3TT2A`, `SM3TT2B`, `SM3PARTW1`, `SM3PARTW2`. |
| SM4 | `SM4E`, `SM4EKEY`. |
| Polynomial multiply | `PMUL`, `PMULL`, `PMULL2`. |

## 30. Pointer authentication, branch protection en geheugenlabels

| Familie | Instructies / werking |
|---|---|
| PAC create | `PACIA`, `PACIB`, `PACDA`, `PACDB` en zero-modifier `PACIZA/B`, `PACDZA/B`. Maakt pointer authentication code. |
| PAC authenticate | `AUTIA`, `AUTIB`, `AUTDA`, `AUTDB` en zero-modifiervormen. |
| Strip PAC | `XPACI`, `XPACD`, `XPACLRI`. |
| Combined load/auth | `LDRAA`, `LDRAB`. |
| Authenticated branch | `BRAA/B`, `BLRAA/B`, zero-modifiervormen, `RETAA/B`. |
| Generic auth | `PACGA` — algemene PAC uit twee waarden. |
| BTI | `BTI c/j/jc` — geldige bestemming voor indirecte branches. |
| Guarded control stack | `GCSPUSHM`, `GCSPOPM`, `GCSPUSHX`, `GCSPOPX`, `GCSSTR`, `GCSSTTR`, `GCSSS1`, `GCSSS2` wanneer GCS aanwezig is. |
| MTE tags | `IRG`, `ADDG`, `SUBG`, `GMI`, `LDG`, `STG`, `STZG`, `ST2G`, `STZ2G`, `STGP`, `LDGM`, `STGM`, `STZGM`. |

## 31. ARM systeemregisters, exceptions en cache/TLB

> Meestal privileged of alleen toegestaan op specifieke Exception Levels.

| Instructie | Werking |
|---|---|
| `MRS` | Leest systeemregister naar algemeen register. |
| `MSR` | Schrijft algemeen register of immediate naar systeemregister/PSTATE-field. |
| `SYS` | Voert algemene systeemoperatie uit, waaronder cache/TLB-vormen. |
| `SYSL` | Systeemoperatie met resultaat. |
| `SVC #imm` | Supervisor call exception. OS-gebruik is ABI-specifiek; syscalltabellen zijn niet opgenomen. |
| `HVC #imm` | Hypervisor call. |
| `SMC #imm` | Secure monitor call. |
| `DC op, Xt` | Data-cache maintenance, bijvoorbeeld clean/invalidate/zero. |
| `IC op, Xt` | Instruction-cache maintenance. |
| `TLBI op, Xt` | TLB-invalidatie. |
| `AT op, Xt` | Address translation probe. |
| `CFP`, `CPP`, `DVP` | Context/cache prediction restrictioninstructies. |

## 32. SVE — Scalable Vector Extension

SVE-instructies werken op implementatie-afhankelijke vectorlengtes van 128 tot 2048 bits en gebruiken predicatie. `/m` betekent merge van inactieve lanes, `/z` maakt ze nul.

| Categorie | Belangrijkste instructiefamilies |
|---|---|
| Predicate maken | `PTRUE`, `PFALSE`, `WHILELT`, `WHILELE`, `WHILELO`, `WHILELS`, `WHILERW`, `WHILEWR`. |
| Predicate logica | `AND`, `ANDS`, `BIC`, `BICS`, `EOR`, `EORS`, `NAND`, `NANDS`, `NOR`, `NORS`, `ORN`, `ORNS`, `ORR`, `ORRS`, `SEL`, `NOT`. |
| Predicate testen | `PTEST`, `PTESTS`, `CNTP`, `PFIRST`, `PNEXT`, `BRKPA`, `BRKPB`, `BRKA`, `BRKB`, `BRKN`. |
| Vectorlengte | `CNTB/H/W/D`, `INCB/H/W/D`, `DECB/H/W/D`, `ADDVL`, `ADDPL`, `RDVL`. |
| Index/generate | `INDEX`, `DUP`, `DUPM`, `CPY`, `MOV`, `SEL`. |
| Integer arithmetic | `ADD`, `SUB`, `SUBR`, `MUL`, `MAD`, `MSB`, `MLA`, `MLS`, `SDIV`, `UDIV`, `SDIVR`, `UDIVR`. |
| Saturating/halving | `SQADD`, `UQADD`, `SQSUB`, `UQSUB`, `SHADD`, `UHADD`, `SHSUB`, `UHSUB`, `SRHADD`, `URHADD`. |
| Widening | `SADDLB/T`, `UADDLB/T`, `SSUBLB/T`, `USUBLB/T`, `SADDWB/T`, `UADDWB/T`, `SSUBWB/T`, `USUBWB/T`. |
| Multiply widening | `SMULLB/T`, `UMULLB/T`, `SMLALB/T`, `UMLALB/T`, `SMLSLB/T`, `UMLSLB/T`. |
| Dot products | `SDOT`, `UDOT`, `SUDOT`, `USDOT` afhankelijk van SVE2-features. |
| Min/max | `SMIN`, `SMAX`, `UMIN`, `UMAX`, reductions `SMINV`, `SMAXV`, `UMINV`, `UMAXV`. |
| Absolute/difference | `ABS`, `NEG`, `SABD`, `UABD`, `SABA`, `UABA`. |
| Compare | `CMPEQ`, `CMPNE`, `CMPGT`, `CMPGE`, `CMPHI`, `CMPHS`, `CMPLE`, `CMPLT`, plus wide varianten. |
| Logic | `AND`, `ORR`, `EOR`, `BIC`, `EON`, `ORN`, `BSL`, `BSL1N`, `BSL2N`, `NBsl`. |
| Shifts | `LSL`, `LSR`, `ASR`, `LSLR`, `LSRR`, `ASRR`, `RSHL`, `SRSHL`, `URSHL`, `SLI`, `SRI`. |
| Bit operations | `CLZ`, `CLS`, `CNT`, `RBIT`, `REV`, `BREV`, `BDEP`, `BEXT`, `BGRP`. |
| Permute | `EXT`, `SPLICE`, `COMPACT`, `TBL`, `TBX`, `ZIP1/2`, `UZP1/2`, `TRN1/2`, `REV*`. |
| Contiguous load/store | `LD1B/H/W/D/Q`, signed extending `LD1SB/SH/SW`, nonfaulting `LDNF1*`, first-fault `LDFF1*`, `ST1B/H/W/D/Q`. |
| Structure load/store | `LD2*`, `LD3*`, `LD4*`, `ST2*`, `ST3*`, `ST4*`. |
| Gather/scatter | `LD1*` en `ST1*` met vector base/offset-indexvormen. |
| Predicate load/store | `LDR Pn`, `STR Pn`; vector `LDR Zn`, `STR Zn`. |
| Prefetch | `PRFB`, `PRFH`, `PRFW`, `PRFD`. |
| Floating arithmetic | `FADD`, `FSUB`, `FSUBR`, `FMUL`, `FDIV`, `FDIVR`, `FMLA`, `FMLS`, `FNMLA`, `FNMLS`, `FNMUL`, `FSQRT`. |
| Floating min/max | `FMIN`, `FMAX`, `FMINNM`, `FMAXNM`, reductions `FMINV`, enz. |
| Floating compare | `FCMEQ`, `FCMNE`, `FCMGT`, `FCMGE`, `FCMLT`, `FCMLE`, `FACGT`, `FACGE`. |
| FP estimates | `FRECPE`, `FRECPS`, `FRECPX`, `FRSQRTE`, `FRSQRTS`. |
| FP round/convert | `FRINT*`, `FCVT*`, `SCVTF`, `UCVTF`. |
| Complex | `FCADD`, `FCMLA`, `CADD`, `CMLA`, `SQRDCMLAH`. |
| Reductions | `ADDV`, `SADDV`, `UADDV`, `FADDV`, `FADDA`, min/max-reductions, `ORV`, `EORV`, `ANDV`. |

## 33. SVE2 en SVE2-uitbreidingen

| Categorie | Families |
|---|---|
| Carry arithmetic | `ADCLB`, `ADCLT`, `SBCLB`, `SBCLT`. |
| Narrowing | `ADDHNB/T`, `SUBHNB/T`, `RADDHNB/T`, `RSUBHNB/T`, `SHRNB/T`, `RSHRNB/T`, saturerende narrowfamilies. |
| Long arithmetic | `SADDLBT`, `SSUBLTB`, mixed signed/unsigned widenfamilies afhankelijk van feature. |
| Crypto AES | SVE2-AES `AESE`, `AESD`, `AESMC`, `AESIMC` vectorlengte-agnostische vormen. |
| Crypto bitperm | `BDEP`, `BEXT`, `BGRP`. |
| SHA3 | `EOR3`, `BCAX`, `RAX1`, `XAR`. |
| SM4 | `SM4E`, `SM4EKEY`. |
| Polynomial | `PMULLB`, `PMULLT`. |
| Saturating multiply | `SQDMULLB/T`, `SQRDMULH`, `SQDMULH`, `SQRDMLAH`, `SQRDMLSH`. |
| Histogram | `HISTCNT`, `HISTSEG`. |
| Match | `MATCH`, `NMATCH`. |
| Table | `TBL`, `TBX` met uitgebreidere vormen. |
| Multi-vector SVE2.1 | Groepen die meerdere opeenvolgende Z-registers behandelen; feature- en assemblerversieafhankelijk. |

## 34. SME — Scalable Matrix Extension

SME voegt streaming SVE mode, het matrixregister `ZA` en in nieuwere versies `ZT0` toe.

| Instructie/familie | Werking |
|---|---|
| `SMSTART` | Activeert streaming mode en optioneel ZA. |
| `SMSTOP` | Deactiveert streaming mode en/of ZA. |
| `ZERO ZA` | Maakt geselecteerde ZA-state nul. |
| `MOVA` / `MOV` | Verplaatst tussen ZA-slices en Z-registers. |
| `LDR ZA`, `STR ZA` | Slaat/herstelt ZA-arraystate wanneer encoding/feature dit biedt. |
| `LD1*`, `ST1*` ZA-slices | Laadt/slaat horizontale of verticale ZA-slices. |
| `FMOPA`, `FMOPS` | Floating outer-product accumulate/subtract naar ZA. |
| `BFMOPA`, `BFMOPS` | BF16 outer-product. |
| `SMOPA`, `SMOPS`, `UMOPA`, `UMOPS`, `SUMOPA`, `SUMOPS`, `USMOPA`, `USMOPS` | Integer outer-products met signed/unsignedcombinaties. |
| `FMLAL`, `FMLSL` SME-vormen | Long floating multiply-accumulate op ZA-slices. |
| `ADDHA`, `ADDVA` | Horizontale/verticale vectoraccumulatie naar ZA. |
| `RDSVL` | Leest streaming vectorlengte-multiple. |
| SME2 multi-vector | `ADD`, `SUB`, `MUL`, `FADD`, `FMUL`, `FMLA`, dot/matrix- en load/storevormen over registergroepen. |
| `LUTI2`, `LUTI4` | Tabellookup via `ZT0` in SME2-uitbreidingen. |

---

# Deel III — Wat bewust niet als syscallreferentie is opgenomen

De volgende zaken zijn **geen normale CPU-rekeninstructies** maar interfaces naar een besturingssysteem of runtime en staan daarom niet als syscalltabel in dit document:

- Linux-syscallnummers zoals `read`, `write`, `openat`, `mmap`, `clone`, `socket` en `io_uring_enter`.
- Windows NT-syscallnummers en Win32 API-functies.
- macOS/BSD-syscallnummers.
- C-libraryfuncties zoals `printf`, `malloc`, `memcpy` en `pthread_create`.
- Assemblerdirectives zoals `section`, `.text`, `.data`, `db`, `dq`, `.byte`, `.global` en macro-opdrachten. Dit zijn opdrachten aan de assembler, geen CPU-instructies.

`SYSCALL`, `SYSENTER`, `SVC`, `HVC` en `SMC` zijn wél architecturale CPU-instructies en zijn daarom kort vermeld, maar hun OS-specifieke functienummers niet.

# Deel IV — Extensies herkennen

## x86-64

Gebruik `CPUID` of een OS/runtimefunctie om te controleren of uitbreidingen zoals AVX2, AVX-512, AES, SHA, BMI of AMX aanwezig en door het OS ingeschakeld zijn. Alleen een moderne assembler gebruiken betekent niet dat de huidige CPU de instructie kan uitvoeren.

## ARM64

ARM64-features worden doorgaans via OS-provided featureinformatie gecontroleerd. De exacte instructies die beschikbaar zijn hangen af van de architectuurversie en optionele features zoals LSE, SVE, SVE2, SME, MTE, PAUTH en crypto.

# Deel V — Officiële bronnen

Voor exacte encodings, pseudocode, exceptions en alle feature-afhankelijke varianten blijven de architectuurhandleidingen de gezaghebbende bron:

1. Intel, **Intel 64 and IA-32 Architectures Software Developer’s Manual, Volumes 2A–2D: Instruction Set Reference A–Z**.
2. AMD, **AMD64 Architecture Programmer’s Manual, Volumes 1–5**.
3. Arm, **A64 Instruction Set Architecture — Base Instructions**, plus de afzonderlijke SIMD, SVE, SVE2 en SME-indexen.

Laatste inhoudelijke controle van de bronfamilies: augustus 2026.

# Snelle vergelijking

| Taak | x86-64 | ARM64 |
|---|---|---|
| Waarde kopiëren | `MOV` | `MOV` alias / `ORR`, `MOVZ`, `MOVK` |
| Geheugen lezen | Vaak `MOV reg,[mem]` | `LDR` |
| Geheugen schrijven | Vaak `MOV [mem],reg` | `STR` |
| Adres maken | `LEA` | `ADR`, `ADRP`, `ADD` |
| Optellen | `ADD` | `ADD` / `ADDS` |
| Vergelijken | `CMP` | `CMP` alias van `SUBS` |
| Branch gelijk | `JE` | `B.EQ` |
| Call | `CALL` | `BL` / `BLR` |
| Return | `RET` | `RET` |
| Atomische CAS | `LOCK CMPXCHG` | `CAS` of `LDXR`/`STXR`-lus |
| SIMD | XMM/YMM/ZMM | V/NEON, Z/SVE |
| Matrix | AMX tiles | SME ZA |

