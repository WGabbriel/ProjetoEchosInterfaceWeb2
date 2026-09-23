# API Echos

Base: `http://localhost:8000`. Tudo JSON. Sem auth, sem CSRF (`@csrf_exempt` na ingestão).
POSTs espelham no Firebase RTDB; GETs `get_dados*` leem da memória do processo.

## Health

```sh
curl -s http://localhost:8000/
# {"status": "ok"}
```

## Ingestão (sensores → POST)

Estações `est0001/0002/0003` → `dados/`, `dados_dois/`, `dados_tres/`. Corpo:

```sh
curl -s -X POST http://localhost:8000/dados/ -H 'Content-Type: application/json' -d \
 '{"Temperatura":25.0,"Pressao":1010.0,"Umidade":60.0,"Gas":1.0,"Rpm":0.0,"Vento":5.0,"Ar":2.0,"Volt":12.0,"Luz":100.0,"Data":"23/09/2026","Hora":"10:00","Chuva_acumulada":0.0}'
# {"status": "ok"}
```

Boias `boia0001/0002/0003` → `dados_quatro/`, `dados_cinco/`, `dados_seis/`. Corpo:

```sh
curl -s -X POST http://localhost:8000/dados_quatro/ -H 'Content-Type: application/json' -d \
 '{"Sensor UV(V)":0.5,"Turbidez(NTU)":1.0,"Tensão(V)":3.3,"ADC":10.0,"Ph":7.0,"Data":"23/09/2026","Hora":"10:00"}'
# {"status": "ok"}
```

Estação `est0004` → `dados_sete/`. Corpo (chaves do firmware):

```sh
curl -s -X POST http://localhost:8000/dados_sete/ -H 'Content-Type: application/json' -d \
 '{"bme_temp":25.0,"bme_pres":1010.0,"bme_hum":60.0,"bme_alt":800.0,"scd_co2":400.0,"Rpm":0.0,"wind_ms":5.0,"scd_temp":24.0,"bus_voltage_v":12.0,"lux":100.0,"Data":"23/09/2026","Hora":"10:00","pluv_mm":0.0,"uv_mv":10.0,"scd_hum":55.0,"current_mA":100.0,"power_mW":1200.0,"mah_total":500.0,"pluv_total_mm":0.0}'
# {"status": "ok"}
```

## Última leitura (React polling → GET)

`get_dados/`, `get_dados_dois/`, `get_dados_tres/`, `get_dados_quatro/`, `get_dados_cinco/`, `get_dados_seis/`, `get_dados_sete/` (1:1 com os POSTs acima).

```sh
curl -s http://localhost:8000/get_dados/
# {"t":25.0,"p":1010.0,"u":60.0,"g":1.0,"r":0.0,"v":5.0,"a":2.0,"vl":12.0,"lz":100.0,"c":0.0}
```

Chaves curtas estação: `t` temp, `p` pressão, `u` umidade, `g` gás, `r` rpm, `v` vento, `a` ar, `vl` volt, `lz` luz, `c` chuva acumulada.
Chaves curtas boia: `u` UV, `tur` turbidez, `ten` tensão, `a` ADC, `p` Ph.
Limite conhecido: antes do primeiro POST após restart, responde 500 (`NameError`, sem inicialização das globals).

## Datas disponíveis (→ GET)

```sh
curl -s "http://localhost:8000/get_dados_armazenados/?estname=est0001"
# {"datas": ["2/9/2026", "3/9/2026", "8/9/2026"]}
```

`estname`: `est0001..est0004` ou `boia0001..boia0003` (sem hífen, minúsculo). Sem dados: `{"datas": "Dados não encontrados"}`.

## Série + estatísticas por período (→ GET `rg/`)

```sh
curl -s "http://localhost:8000/rg/?datadados=2/9/2026&datadadosdois=8/9/2026&estdados=est-0001"
```

Params: `datadados` (início `d/m/Y`), `datadadosdois` (fim), `estdados` (`est-000X`/`boia-000X`, com hífen).
Sucesso: `series` (`leitura` + valores na mesma ordem) + última leitura, médias (`*Med`), máx/mín (`*Max/*Min`), `DataValida: true`.
Sem dados no período: `{"DataInvalida": true, "DataValida": false}`.
