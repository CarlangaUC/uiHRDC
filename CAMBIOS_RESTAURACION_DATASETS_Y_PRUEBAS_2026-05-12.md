# Restauracion de datasets y pruebas (2026-05-12)

Este documento complementa:

- `/root/MAGISTER/CAMBIOS_CONVERSACION_VERSIONADO_ZDD_2026-05-11.md`
- `/root/MAGISTER/CAMBIOS_SESION_ZDD_BITPACKING_PFD.md`

## Objetivo

1. Recuperar datasets de trabajo despues del reset.
2. Regenerar el entorno de pruebas en `uiHRDC/uiHRDC/data/texts`.
3. Probar build en orden: `wiki_100mb`, `torsen.text200mb`, `wiki_2gb`.
4. Preparar cambios para commit/push sin incluir datasets pesados.

## Restauracion de datasets

### Origen de datos

- Se extrajeron archivos desde:
  - `uiHRDC/uiHRDC/data/texts/text2gb.7z` (contiene `wiki_src2gb.*`)
  - `uiHRDC/uiHRDC/data/texts/text200mb.7z` (contiene `torsen.text200mb.*`)
- Se usaron/copiaron datasets desde `scripts/archivos_test`:
  - `wiki_100mb.txt`, `wiki_100mb.txt.DOCBOUNDARIES.ul`, `wiki_100mb.log`
  - `wiki_2gb.txt`, `wiki_2gb.txt.DOCBOUNDARIES.ul`, `wiki_2gb.log`
  - `torsen.text200mb.txt`, `torsen.text200mb.txt.DOCBOUNDARIES.ul`

### Mappings versionados

- Copiados a `uiHRDC/uiHRDC/data/texts`:
  - `page_mapping_wiki_100mb.bin`
  - `page_mapping_wiki_2gb.bin`

## Ajustes de compilacion necesarios

Para compilar con `g++13` en este entorno se reaplicaron fixes de compatibilidad:

- `uiHRDC/uiHRDC/indexes/NOPOS/II_docs/src/utils/basics.h`
  - `byte` via `typedef` protegido.
  - `#undef min` / `#undef max` en C++.
- `uiHRDC/uiHRDC/indexes/NOPOS/II_docs/src/utils/defValues.h`
  - `byte` via `typedef` protegido.
- `uiHRDC/uiHRDC/indexes/NOPOS/II_docs/src/utils/hashWords.h`
  - eliminada redefinicion local de `byte`.
- `uiHRDC/uiHRDC/indexes/NOPOS/II_docs/Makefile`
  - `CFLAGS` con `-no-pie` para evitar error de linker PIE.

## Compilacion

Directorio:

- `uiHRDC/uiHRDC/indexes/NOPOS/II_docs`

Comandos:

- `export TEXT_COMPRESSOR=NOTEXT INDEX_TYPE=PFORDELTA`
- `make clean`
- `make buildII`

Resultado: **OK** (solo warnings legacy `register`).

## Pruebas de build por dataset

### 1) wiki_100mb (versionado esperado)

Comando:

- `./BUILD_PFORDELTA_NOTEXT .../wiki_100mb.txt .../index_wiki_100mb_named "nooptions"`

Resultado:

- Build completo **OK**.
- Log reporta:
  - `build metadata: versioned postings enabled`
  - `Versioned postings enabled = 1`
  - `Version map entries = 15`

### 2) torsen.text200mb (no versionado esperado)

Comando:

- `./BUILD_PFORDELTA_NOTEXT .../torsen.text200mb.txt .../index_torsen_200mb_named2 "nooptions"`

Resultado:

- Build completo **OK**.
- Log reporta:
  - warning por falta de `page_mapping_torsen.text200mb.bin`
  - `build metadata: no mapping found`
  - `Versioned postings enabled = 0`
  - `Version map entries = 0`

### 3) wiki_2gb (versionado esperado)

Comando:

- `II_SKIP_BUILD_VERIFY=1 ./BUILD_PFORDELTA_NOTEXT .../wiki_2gb.txt .../index_wiki_2gb_named "nooptions"`

Resultado:

- Avanza correctamente por:
  - carga de texto,
  - 1st pass,
  - 2nd pass,
  - `prepareSourceFormatForIListBuilder`,
  - entrada a `BUILD INVERTED LIST REPRESENTATION`.
- La corrida no finalizo en la ventana de trabajo y fue detenida manualmente para continuar con la preservacion de cambios.

## Politica de commit/push

- No incluir en commit:
  - datasets (`*.txt` grandes, `*.DOCBOUNDARIES*`, `page_mapping_*.bin`),
  - indices generados (`*.const`, `*.voc`, `*.il.*`),
  - logs de ejecucion.
- Solo incluir codigo fuente y documentacion.
