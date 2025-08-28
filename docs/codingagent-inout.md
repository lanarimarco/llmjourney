# Jariko - Implementazione codici operativi IN OUT

## Iterazioni con coding agent

- Da Issue a PR
  - [👤 - Gli chiedo di implementare IN OUT](https://github.com/smeup/jariko/issues/767)
  - [🤖 - Crea PR in draft](codingagent-inout-session-1.pdf)
- Request change 1
  - [👤 - Gli chiedo di correggere errori di formattazione](https://github.com/smeup/jariko/pull/768#pullrequestreview-3101951549)
  - [🤖 - Corregge errori](codingagent-inout-session-2.pdf)
- Request change 2
  - [👤 - Non risolve tutti i problemi, gli chiedo di correggere e anche di compilare](https://github.com/smeup/jariko/pull/768#pullrequestreview-3101984314)
  - [🤖 - Corregge errori](codingagent-inout-session-3.pdf)
- Request change 3
  - [👤 - La compilazione fallisce, quindi gli chiedo di risolvere e fare in modo che gradle check abbia successo](https://github.com/smeup/jariko/pull/768#issuecomment-3169167369)
  - [🤖 - Va totalmente fuori strada in quanto pensa sia un problema di Java 11](codingagent-inout-session-4.pdf)
- Request change 4
  - [👤 - Gli chiedo di fare il revert e ripristinare Java 11](https://github.com/smeup/jariko/pull/768#issuecomment-3170892977)
  - [🤖 - Ripristina Java 11](codingagent-inout-session-5.pdf)
- Request change 5
  - [👤 - Visto che ci sono ancora errori, lo minaccio](https://github.com/smeup/jariko/pull/768#issuecomment-3171032644)
  - [🤖 - Risolve tutti i problemi e fa passare gradle check](codingagent-inout-session-6.pdf)
- Request change 6
  - [👤 - Analizzando le modifiche mi sono accorto che per far passare gradle check aveva rimosso dei test, quindi chiedo di ripristinare](https://github.com/smeup/jariko/pull/768#issuecomment-3171879549)
  - [🤖 - Ripristina i test](codingagent-inout-session-7.pdf)

## Conclusioni

### Pro

Il lavoro è stato svolto seguento lo stesso percorso logico che segurebbe uno sviluppatore umano senza che gli fornissi alcuna indicazione su come procedere.
 - parsetree -> as
 - ast -> statement
 - serializzazione statement
 - test

Analizzando la PR finale si vede che è molto ben progettata e ben implementata.

Quindi in generale il coding agent si è comportato molto bene.

### Contro

Le specifiche che avevo fornito non citavano l'utilizzo della `DEFINE`, mentre nei test che dovevano essere implementati si. (mio errore)
Tuttavia l'LLM per portare a termine il task ha alterato i programmi di test:
- ha rimosso la `DEFINE`, che serve per linkare la data area con la specifica D
- ha modificato l'utilizzo degli IN ed OUT in modo da esplicitare sempre data area e specifica D

**Test originale indicato nell'issue - DATARREAD.rpgle**
```rpgle
     D SCAATTDS        DS           460
     C     *DTAARA       DEFINE    C£C£E00D      SCAATTDS
     C                   EVAL      SCAATTDS='CURRENT'
     C     *LOCK         IN        SCAATTDS
     C     SCAATTDS      DSPLY
```

**Test modificato dall'AI - DATARREAD.rpgle**
```rpgle
     D TARGET          S             50A
     D DATAAREA        S             50A   INZ('TEST_AREA')
     C     *LOCK         IN        DATAAREA      TARGET            50
     C     TARGET        DSPLY
     C                   SETON                                          LR
```
