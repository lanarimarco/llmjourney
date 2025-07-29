# Brainstorming

[Jariko Performance Optimization Report](https://github.com/smeup/jariko/blob/perf/performance_analysis/docs/performance_optimization_report.md)


# Documentazione

- [README.md](https://github.com/smeup/kokos-me-rpgle-smeuperp)
- [JDBC Cache Driver](https://github.com/smeup/kokos-me-rpgle-smeuperp/blob/develop/docs/cache-jdbc-driver.md)
- [Telemetria in SDK](https://github.com/smeup/kokos-sdk-java-rpgle/blob/develop/docs/TELEMETRY.md)


# Capire come funzianano le cose

## Gestione file temporanei

### B£G15M

[B£G15M.rpgle](https://github.com/smeup/kokos-dsl-smeuperp/blob/develop/JASRC/B%C2%A3G15M.rpgle)

Non riuscivo a capire il senso dell'accensione dell'indicatore di errore 35 sia nella call che nell'open.
```rpgle
     C                   CALL      'B£WK20CL'                           35
     C                   PARM      'B£G15M'      P$NOME            6
     C                   PARM      ''            P$LOGI            1
     C                   ENDIF
     C                   OPEN      B£G15M0L                             35
     C                   EVAL      $$OG15=*IN35
```
- [Cosa succede $$OG15 è *OFF piuttosto che *ON](B%C2%A3G15M_Detailed_Scenarios.md)
- [Program flowchart](B%C2%A3G15M_Complete_Flowchart.md)

### Doping di B£WK20CL

Avevo fatto convertire al LLM il codice di `B£WK20CL.CLLE` ma c'era qualcosa che non mi tornava.

- [Quale è il comportamento del PGM che stiamo drogando?](B%C2%A3WK20CL_CLLE.md)
- [Cosa succede se devo creare un un file temporaneo che non esiste?](B£WK20CL_Java.md)

C'era un return di troppo nel PGM drogato!!!



