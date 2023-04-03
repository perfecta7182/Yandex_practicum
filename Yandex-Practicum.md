```python
import pandas as pd
from sqlalchemy import create_engine

db_config = {'user': 'challenge_user', # имя пользователя
             'pwd': 'aT00GAh944YU4Q1J4zbzryW6Yrz11iq54coz', # пароль
             'host': 'rc1a-eqy3nymy2wp6tbx2.mdb.yandexcloud.net', # адрес сервера
             'port': 6432, # порт подключения
             'db': 'data-analyst-challenge'}# название базы данных
 
# Формируем строку соединения с БД.
connection_string = 'postgresql://{}:{}@{}:{}/{}'.format(db_config['user'],
                                             db_config['pwd'],
                                             db_config['host'],
                                             db_config['port'],
                                             db_config['db'])
# Подключаемся к БД.
engine = create_engine(connection_string)

# Формируем sql-запрос(пример).
query = ''' WITH cte AS (
  SELECT 
    finished_lesson_test.lesson_id, 
    user_id, 
    profession_id, 
    profession_name, 
    lesson_name, 
    date_created, 
    LAG(date_created) OVER (PARTITION BY user_id, profession_id ORDER BY date_created) AS prev_lesson_date
  FROM 
    finished_lesson_test 
    JOIN lesson_index_test ON finished_lesson_test.lesson_id = lesson_index_test.lesson_id 
  WHERE 
    lesson_index_test.profession_name = 'data-analyst' 
    AND EXTRACT(year FROM date_created) = 2020 
    AND EXTRACT(month FROM date_created) = 4
)
SELECT 
  EXTRACT(SECOND FROM (date_created - prev_lesson_date)) as delta_seconds,
prev_lesson_date,
  date_created, 
  lesson_id, 
  profession_name, 
  user_id
FROM 
  cte 
WHERE 
  date_created - prev_lesson_date <= INTERVAL '5 seconds';

        '''

# Выполняем запрос и сохраняем результат
# выполнения в DataFrame.
# Sqlalchemy автоматически установит названия колонок
# такими же, как у таблицы в БД.
data_raw = pd.io.sql.read_sql(query, con = engine)

```


```python
data_raw.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>delta_seconds</th>
      <th>prev_lesson_date</th>
      <th>date_created</th>
      <th>lesson_id</th>
      <th>profession_name</th>
      <th>user_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4.357835</td>
      <td>2020-04-16 14:41:54.781518+00:00</td>
      <td>2020-04-16 14:41:59.139353+00:00</td>
      <td>3891bd65-5ff9-4355-b886-f5796ce6ba72</td>
      <td>data-analyst</td>
      <td>4409</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.457578</td>
      <td>2020-04-12 13:27:55.500944+00:00</td>
      <td>2020-04-12 13:27:59.958522+00:00</td>
      <td>cec6f205-490c-4e36-8167-53db63585876</td>
      <td>data-analyst</td>
      <td>16144</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.515751</td>
      <td>2020-04-12 13:27:59.958522+00:00</td>
      <td>2020-04-12 13:28:04.474273+00:00</td>
      <td>b1505742-72ef-456f-b3f6-932461fadce6</td>
      <td>data-analyst</td>
      <td>16144</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.073750</td>
      <td>2020-04-12 13:28:04.474273+00:00</td>
      <td>2020-04-12 13:28:08.548023+00:00</td>
      <td>2a608a6c-97a2-45e2-996a-a215c15756e8</td>
      <td>data-analyst</td>
      <td>16144</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3.927094</td>
      <td>2020-04-12 13:36:33.836217+00:00</td>
      <td>2020-04-12 13:36:37.763311+00:00</td>
      <td>d8447ed8-0b62-4107-8b6a-10ce6fd8fbd7</td>
      <td>data-analyst</td>
      <td>16144</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Правда непонятно почему в задании указано вывести lesson_id

unique_values = data_raw['lesson_id'].nunique()
print('Кол-во уроков с быстрым прохождением')
print(unique_values)

import plotly.express as px
# проверяем, есть ли уроки с слишком быстрым временем прохождения
if data_raw.empty:
    print('Нет уроков с слишком быстрым временем прохождения')
else:
    # считаем количество таких уроков
    num_fast_lessons = len(data_raw)
    # выводим наименования уроков
    fast_lesson_ids = list(data_raw['lesson_id'].unique())
    print('Есть уроки с слишком быстрым временем прохождения:')
    print(fast_lesson_ids)
    # создаем интерактивный график для визуализации распределения времени прохождения уроков
    fig = px.histogram(data_raw, x='delta_seconds', nbins=30,
                       title='Распределение времени прохождения уроков')
    fig.show()
```

    Кол-во уроков с быстрым прохождением
    91
    Есть уроки с слишком быстрым временем прохождения:
    ['3891bd65-5ff9-4355-b886-f5796ce6ba72', 'cec6f205-490c-4e36-8167-53db63585876', 'b1505742-72ef-456f-b3f6-932461fadce6', '2a608a6c-97a2-45e2-996a-a215c15756e8', 'd8447ed8-0b62-4107-8b6a-10ce6fd8fbd7', '47cf9db0-4be1-412e-80f7-a3971b6b3c88', 'd0162814-ae0a-4c0c-8e60-22ecf53c5348', 'c43465c3-1579-43d2-b229-0c0eaec20e27', '21fbfb7c-8b74-40e7-9cd8-9590b7903f26', '2fb8f1df-6e9a-4b1f-86ff-ca213ef98cc7', 'dbb9697d-3d32-4b81-ad1c-c9ac0496ee39', '6a4be97b-048a-43d8-b8f0-964f52ac7d30', '24d355f6-4f88-4c05-bf51-e0e1db42f184', 'cd601912-2c45-429c-b9f1-3ef16fb07a31', 'a7b2c524-eef0-46af-b065-044492a4f4c0', 'e1324af1-8285-4c22-93f0-10e7641dc254', 'cc57bb5d-707b-4466-8c6b-651d9447dfe5', '83b099db-0830-4bc6-b6ec-fe25a29a4e42', '35bef149-e188-4385-8622-2c73911dfc86', '86fbb98c-0003-4cdd-80cd-55fadc110334', 'efcdf907-60ff-41d2-aacb-7789d9c2f443', 'ba6241c8-28da-4035-863f-400500d397e7', '19c16deb-5db3-43d7-96c3-49ee749c0649', '55db68d2-3657-422c-aa60-099db8f8e484', '5a0b77b8-f9cb-4413-bbbc-1a3b5e68861e', '7c171fe6-939e-48fd-8588-e379b6037360', '90f1fe45-2e98-4144-aafb-a07b886b26f5', 'ad715968-32ae-4bcb-bcf8-4f9908cad270', 'f47f257b-9f50-4221-91b1-b50b536ed82f', '0f82b02e-0598-4c2d-8921-c04d73ea8827', 'e6f68e73-9a82-4025-8cf0-14fb4f76c034', 'c2e24cf4-94fc-4570-b609-6a570f13b0ea', '27039113-5c1f-4175-bdef-d42be07068e9', 'd2f4b6b4-e76f-4203-b30e-1f6d1bcc7262', 'eda598a9-1dd1-46b4-80bd-06cad4cd23b3', '36112acf-454a-4174-a0ce-f20a019f9cf4', '3c365926-e395-4e7c-9d7a-4fb97bb145f9', 'e52c9904-6dba-4822-b0a1-b4ec8ab99853', '07a65ccc-e8ef-4148-9657-aea08b0e1930', 'e3147841-688f-44ca-91b1-e0db2c9e3faf', 'd2909b55-6bea-4e59-8be2-00ad23c11821', 'a550ea40-7783-46cf-aba8-732ff19df7d6', '4a2a2193-2d0a-4669-89fa-c20e8e117840', 'a341596d-0793-4333-a3b5-0eb24f7f6b77', '0d7855ec-841c-4ce8-bad7-50a404f59fc9', '48537cd0-2405-438a-ac7b-d79d3d776931', '8866e43f-260d-41c8-b2b9-91fc14e858f4', '6bb4e45e-1155-4887-999e-79506bc2e444', '1cd0f147-a523-42ca-ac77-b6b0ca251007', 'bb3ff463-8ada-4850-86a1-3b0ebca17488', 'f38e4b6d-d961-4b5f-9d77-39588cc18d7b', 'fb435331-b60a-4e4b-823f-65028f92ec91', 'b0cc1d7e-dd9f-499b-aa3b-a4a287a33b42', '552800a4-f5af-4171-ad86-eb740ad0dd6f', '367df9a0-7679-46b0-ab69-9efa2bdaa639', 'd61dc3cd-4499-4186-b914-cd4709948966', 'be1011a7-d5e9-4e21-8120-acabc1d29ad9', 'a446a781-2ba2-42a5-9db1-5e3a8052565a', '01c5aa7d-ce7d-4a1c-9da2-7945f025c2eb', 'd71a4212-73d7-4b8c-bd43-d3d3381af97d', 'caf07b68-dc35-4ebd-962b-a8b2794b1e4f', '5962f265-61b1-4007-87de-d1132d98530c', 'e08c3d4a-a46b-4775-9f05-5fd94cac53a0', 'cbadcca3-dcca-4257-b343-01133a1b96c9', 'e08ec00b-2bf1-449a-ba94-5e1dc6fd948b', 'a9c9b940-ac6e-4c12-89ee-b3fd0f90cf17', '3836b9b3-c584-4009-91c8-947f578d2327', 'da902a3f-83a1-4a8b-893e-da689cec44a5', '3b9c215b-e736-4150-be01-33086b0129f2', 'cf77fb0e-4854-40df-86e2-730e01fbbc67', '94639c14-f7e7-4e66-a037-dd66600ec2e7', 'a74863bb-85ac-437b-a334-f2bc456e31b4', '24e1ee92-ca4f-424b-95b2-b941a278af0a', 'b65b6ada-b087-44ba-b66d-74a05b96e8b7', 'a8bfc93d-27b9-4bc5-a20c-aca067ee63b1', 'ab46ee96-5b67-4259-84ee-7d5e17f0cb25', '98397c15-7e3f-493f-a851-347704f35663', '04e90fe7-376d-4d93-a460-a204288b1a13', 'be47a841-c0c7-4131-95ae-a6ff4845227e', '792a07d1-74c4-4c58-8275-b5df3f1cdb5c', '7f822db5-7360-4f5c-b452-75b25aa3d658', 'c526a357-eb40-4d6c-b9be-61e24b9b6e4a', '7fef6e95-d8ec-40aa-b290-9e73d9d5cf0d', '2e49d062-d360-47e4-ab55-887c8e79d7d9', 'a96dacf0-467c-4e60-940c-d119c6e5a96b', '9a2be4f6-1e7e-4630-9530-3d1f7939e429', '9a9a4ead-8c15-4efd-9e33-d691aa9c28ef', 'e651845e-430c-4123-a5e7-eb320455accf', '48edcfc8-4e69-445f-9a27-a9b6e296caf6', '0d0b0084-5e94-41d2-b31a-2d3e56913f6f', '79aa5d37-fa47-4342-a9a3-d756d74e4540']
    


        <script type="text/javascript">
        window.PlotlyConfig = {MathJaxConfig: 'local'};
        if (window.MathJax && window.MathJax.Hub && window.MathJax.Hub.Config) {window.MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
        if (typeof require !== 'undefined') {
        require.undef("plotly");
        define('plotly', function(require, exports, module) {
            /**
* plotly.js v2.20.0
* Copyright 2012-2023, Plotly, Inc.
* All rights reserved.
* Licensed under the MIT license
*/
/*! For license information please see plotly.min.js.LICENSE.txt */
        });
        require(['plotly'], function(Plotly) {
            window._Plotly = Plotly;
        });
        }
        </script>




<div>                            <div id="f44c490e-0652-4f8a-b925-ec40bd880ced" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("f44c490e-0652-4f8a-b925-ec40bd880ced")) {                    Plotly.newPlot(                        "f44c490e-0652-4f8a-b925-ec40bd880ced",                        [{"alignmentgroup":"True","bingroup":"x","hovertemplate":"delta_seconds=%{x}<br>count=%{y}<extra></extra>","legendgroup":"","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"","nbinsx":30,"offsetgroup":"","orientation":"v","showlegend":false,"x":[4.357835,4.457578,4.515751,4.07375,3.927094,3.357475,3.385988,4.195492,2.845297,4.806588,4.19289,4.618826,4.268954,4.388719,4.120511,4.666201,6.7e-05,3.5e-05,3.2e-05,3.2e-05,3.3e-05,4.914826,3.223666,4.734494,4.130122,4.907058,4.160943,4.430991,4.035356,2.89539,4.972451,4.390571,4.7312,4.836486,3.816613,4.525326,4.301271,4.49767,3.7489,3.738068,2.913689,4.931275,3.657484,4.914156,6.7e-05,3.3e-05,3.2e-05,4.8e-05,4.2e-05,3.9e-05,4.5e-05,3e-05,3e-05,2.9e-05,3e-05,2.9e-05,2.9e-05,2.9e-05,2.9e-05,4.2e-05,3e-05,2.8e-05,2.9e-05,2.8e-05,4.3e-05,3e-05,4.4e-05,3e-05,3e-05,3e-05,3e-05,3.5e-05,3e-05,3.1e-05,3e-05,3e-05,3e-05,3e-05,3e-05,4.4e-05,4.3e-05,3.3e-05,2.9e-05,2.8e-05,2.8e-05,2.8e-05,2.8e-05,4.3e-05,3e-05,4.3e-05,3.1e-05,3.1e-05,3e-05,3.1e-05,2.9e-05,4.4e-05,3e-05,2.9e-05,2.9e-05,4.3e-05,2.8e-05,4.3e-05,4.3e-05,3.2e-05,3e-05,3e-05,4.3e-05,3e-05,3e-05,2.9e-05,4.2e-05,2.9e-05,4.2e-05,2.8e-05,2.8e-05,2.7e-05,4.2e-05,2.9e-05,2.7e-05,2.8e-05,2.8e-05,2.8e-05,2.8e-05,2.9e-05,2.8e-05,2.8e-05,4.1e-05,2.8e-05,4.1e-05,2.8e-05,2.8e-05,2.8e-05,4e-05,3.704891,3.814213,3.280435,4.599478,4.797113,4.704272,3.612467,3.357471,4.555093,4.648038,4.458042,3.554843,3.309941,4.554454,4.562714,3.358285,4.452447,4.178449,3.98088,4.482578,4.293511,4.96504,3.940534,4.46059,4.49593,3.313348,3.503363,2.833377,3.872521,4.978725,4.510533,3.886845,4.841633,2.920791,4.444173,3.864433,3.978351,4.003696,4.484696,4.332806,4.542004,4.260808,4.592885,4.2608,4.569718,3.650399,4.3879,2.690576,3.102318,3.216457,3.822468,3.962959,4.253614,4.773827,4.471869,4.658515,4.170457,3.668398,4.418323,4.749636,4.997253,4.608797,3.640757,4.882766,3.913593,3.538278,4.987526,4.098135,4.876767,4.440573,3.412142,4.954,2.594752,3.72469,4.030403,4.280372,4.303423,4.87963,4.514233,3.605818,4.501813,4.782699,4.070566,4.722162,4.424833,3.3271,3.80275,3.198903,3.497647,3.036466,4.110381,3.377589,3.547107],"xaxis":"x","yaxis":"y","type":"histogram"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"delta_seconds"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"count"}},"legend":{"tracegroupgap":0},"title":{"text":"\u0420\u0430\u0441\u043f\u0440\u0435\u0434\u0435\u043b\u0435\u043d\u0438\u0435 \u0432\u0440\u0435\u043c\u0435\u043d\u0438 \u043f\u0440\u043e\u0445\u043e\u0436\u0434\u0435\u043d\u0438\u044f \u0443\u0440\u043e\u043a\u043e\u0432"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('f44c490e-0652-4f8a-b925-ec40bd880ced');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>
