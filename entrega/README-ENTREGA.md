
# ENTREGA

Ao executar o comando `pytest -q` eu econtrei e corrigi os seguintes 5 problemas encontrados nos testes.

## test_currency_conversion_to_brl

Ao investigar que o teste entregava o resultado 118.45 ao inves do esperado 642.00, descobri que o erro ocorria dentro da função `convert_to_brl()` dentro do serviço `currency_converter.py` porque a operação retonada calculava a converção de dinheiro de forma errada (amount / rate), com isso eu corrigi o seu retorno para `return amount * rate`

```
tmp_path = WindowsPath('C:/Users/gabri/AppData/Local/Temp/pytest-of-gabri/pytest-30/test_currency_conversion_to_br0')

  

def test_currency_conversion_to_brl(tmp_path):

"""Multiplica o valor pela taxa, sem inverter a operação."""

report = run_cli(

tmp_path,

[

{"reservation_id": "R-1", "currency": "BRL", "amount": "100.00"},

{"reservation_id": "R-2", "currency": "USD", "amount": "100.00"},

],

)

> assert report["summary"]["total_amount_brl"] == "642.00"

E AssertionError: assert '118.45' == '642.00'

E

E - 642.00

E + 118.45

  

tests\test_currency.py:15: AssertionError
```

## test_nights_across_month_boundary

Esse problema ocorria no serviço `stay_calculator.py` poque a função `calculate_room_nights()` fazia o calculo de noites de quarto sem considerar o mês do objeto `datev`

```
((checkout.days - checkin.days)* rooms)
```

Ao corrigir para

```
(checkout - checkin).days * rooms
```
O python subtrai uma data da outra e retorna a diferença em dias.

```
tmp_path = WindowsPath('C:/Users/gabri/AppData/Local/Temp/pytest-of-gabri/pytest-30/test_nights_across_month_bound0')

  

def test_nights_across_month_boundary(tmp_path):

"""A diferença de datas considera a virada de mês, não o dia do mês."""

report = run_cli(

tmp_path,

[

{

"reservation_id": "R-1",

"checkin_date": "2025-01-30",

"checkout_date": "2025-02-02",

"rooms": "1",

}

],

)

> assert report["summary"]["total_room_nights"] == 3

E assert -28 == 3

  

tests\test_nights.py:19: AssertionError
```

## test_regression_report_on_sample

Atraves de investigações, eu decobri que esse erro tem a mesma origem e solução que o erro apresentado no log `test_currency_conversion_to_brl`, se tratando do mesmo erro de converção de dinheiro.

```
tmp_path = WindowsPath('C:/Users/gabri/AppData/Local/Temp/pytest-of-gabri/pytest-30/test_regression_report_on_samp0')

  

def test_regression_report_on_sample(tmp_path):

"""Relatório completo de uma amostra pequena, com status, noites e câmbio."""

report = run_cli(

tmp_path,

[

{

"reservation_id": "R-1",

"amount": "1000.00",

"checkin_date": "2025-01-10",

"checkout_date": "2025-01-13",

},

{"reservation_id": "R-2", "status": "CANCELLED", "amount": "500.00"},

{

"reservation_id": "R-3",

"currency": "USD",

"amount": "100.00",

"checkin_date": "2025-01-20",

"checkout_date": "2025-01-22",

},

],

)

> assert report["summary"] == {

"total_reservations": 3,

"confirmed_reservations": 2,

"cancelled_reservations": 1,

"no_show_reservations": 0,

"total_room_nights": 5,

"total_amount_brl": "1542.00",

"average_nightly_rate_brl": "308.40",

}

E AssertionError: assert {'average_nig...ions': 0, ...} == {'average_nig...ions': 0, ...}

E

E Omitting 5 identical items, use -vv to show

E Differing items:

E {'average_nightly_rate_brl': '203.69'} != {'average_nightly_rate_brl': '308.40'}

E {'total_amount_brl': '1018.45'} != {'total_amount_brl': '1542.00'}

E Use -v to get more diff

  

tests\test_regression_report.py:27: AssertionError
```

## test_status_rules_exclude_cancelled_and_no_show

A função `is_billable()` do serviço `status_policy.py` estava indicando nos totais financeiros qualquer valor do parâmetro `status` que seja diferente do valor `"CANCELLED"`, para corrigir esse problema eu substitui a comparação para `return status != "CANCELLED" `

```
tmp_path = WindowsPath('C:/Users/gabri/AppData/Local/Temp/pytest-of-gabri/pytest-30/test_status_rules_exclude_canc0')

  

def test_status_rules_exclude_cancelled_and_no_show(tmp_path):

"""Somente CONFIRMED entra nos totais financeiros."""

report = run_cli(

tmp_path,

[

{"reservation_id": "R-1", "status": "CONFIRMED", "amount": "100.00"},

{"reservation_id": "R-2", "status": "CANCELLED", "amount": "50.00"},

{"reservation_id": "R-3", "status": "NO_SHOW", "amount": "20.00"},

],

)

assert report["summary"]["total_reservations"] == 3

assert report["summary"]["confirmed_reservations"] == 1

assert report["summary"]["cancelled_reservations"] == 1

assert report["summary"]["no_show_reservations"] == 1

> assert report["summary"]["total_amount_brl"] == "100.00"

E AssertionError: assert '120.00' == '100.00'

E

E - 100.00

E ? ^

E + 120.00

E ? ^

  

tests\test_status_rules.py:20: AssertionError
```

## test_top_hotels_are_ranked_descending

De acordo com o que foi demonstrado nos logs, a lista do ranking do serviço `report_builder.py` que deveria mostra os hotéis de maior valor primeiro estava ao contrario `ranked_hotels = sorted(hotels.items(), key=lambda item: (item[1]["amount"], item[0]))` o sistema deu o retorno esperado apos essa coreção `ranked_hotels = sorted(hotels.items(), key=lambda item: (-item[1]["amount"], item[0]))`

  ```
tmp_path = WindowsPath('C:/Users/gabri/AppData/Local/Temp/pytest-of-gabri/pytest-30/test_top_hotels_are_ranked_des0')

  

def test_top_hotels_are_ranked_descending(tmp_path):

"""O ranking mostra os hotéis de maior valor primeiro."""

report = run_cli(

tmp_path,

[

{"reservation_id": "R-1", "hotel_id": "H01", "hotel_name": "Hotel A", "amount": "1000.00"},

{"reservation_id": "R-2", "hotel_id": "H02", "hotel_name": "Hotel B", "amount": "900.00"},

{"reservation_id": "R-3", "hotel_id": "H03", "hotel_name": "Hotel C", "amount": "800.00"},

{"reservation_id": "R-4", "hotel_id": "H04", "hotel_name": "Hotel D", "amount": "700.00"},

],

top=3,

)

> assert [hotel["hotel_id"] for hotel in report["top_hotels"]] == ["H01", "H02", "H03"]

E AssertionError: assert ['H04', 'H03', 'H02'] == ['H01', 'H02', 'H03']

E

E At index 0 diff: 'H04' != 'H01'

E Use -v to get more diff

  

tests\test_top_hotels.py:18: AssertionError
```
  
 
## Leitura Completa do Repositório de Reservas

-  **Problema:** O sistema limitava a leitura do arquivo a apenas 200 linhas.

-  **Causa Raiz:** Havia uma condição de parada antecipada (`if len(rows) == 200: break`) em `csv_reservation_repository.py`.

-  **Correção:** Removida a trava para que o `for` leia o arquivo CSV até o fim, garantindo a contagem correta de todas as reservas do dataset.
  
  

