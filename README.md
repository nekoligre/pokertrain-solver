# pokertrain-solver

Calcula estrategias de equilibrio de póker con [TexasSolver](https://github.com/bupticybee/TexasSolver),
repartiendo el trabajo entre varios runners de GitHub Actions.

Resolver un spot con TexasSolver tarda unos cuatro minutos con cuatro hilos, y
cada spot es independiente del resto. Un ordenador los hace de uno en uno; veinte
runners en paralelo se comen la cola veinte veces antes.

## Qué hay aquí

- `.github/workflows/resolver.yml` — el workflow. Se lanza a mano desde la
  pestaña Actions, nunca solo.
- `tanda/000`, `tanda/001`, … — un fichero de texto por spot, con los comandos
  que entiende `console_solver`. Cada carpeta la coge un runner distinto.

Un fichero de entrada es esto, y nada más:

```
set_pot 100
set_effective_stack 173
set_board 8s,8d,3c
set_range_ip QQ,KK,AA,A4s,A5s,AKs,AKo
set_range_oop JJ,QQ,AKo,AQs
set_bet_sizes oop,flop,bet,33,50,75
...
start_solve
```

Un bote, un stack, un board y dos rangos. No hay historiales de manos, ni
nombres de jugadores, ni cartas de nadie.

## Cómo se usa

1. Poner los ficheros en `tanda/` y empujarlos.
2. Actions → **Resolver tanda** → Run workflow, diciendo cuántos trozos hay.
3. Cuando acabe, bajar los artefactos: `gh run download <run>`.

El resultado de cada spot sale con el mismo nombre que su entrada, en `.json`.
Ese volcado trae la estrategia por combinación en cada nodo del árbol.

## Detalles que importan

**El binario es el oficial.** Se baja la release `v0.2.0` de Linux de
TexasSolver en cada ejecución. Es la misma versión que se usa en local, así que
las estrategias que devuelve son las mismas.

**Los hilos se ajustan al runner.** El fichero de entrada trae `set_thread_num 4`
porque es lo que da un runner normal, pero el workflow lo reescribe con `nproc`.

**Cada spot tiene veinte minutos.** Un árbol que se desmadra no puede comerse
las seis horas que dura un job. Si un spot se pasa de ahí, se descarta y el
resto sigue.

**Los trozos son independientes.** `fail-fast: false`, así que si uno falla los
demás terminan igual.

## Licencia

TexasSolver es de [bupticybee](https://github.com/bupticybee/TexasSolver) y tiene
su propia licencia. Aquí solo hay ficheros de configuración y un workflow.
