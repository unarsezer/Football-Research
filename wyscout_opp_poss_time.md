# **How to calculate opponent's pure possession time in Wyscout**
<hr>

<p style="line-height: 1.5; font-size: 16px;">I was reading the Wyscout Glossary and I came across Possession Adjusted. It's not like I didn't already know that the defensive metrics “per 90 minutes” was wrong and incomplete, but then another light went on in my head. The source of this light was the following sentences:<br><br><em>In the match of AFC Bournemouth - Manchester City (0:1, 2 March 2019) Man City had had 80% of possession (42:37 pure possession time), while Bournemouth only had 20% (10:35 pure possession time).<br>Aké, who played the whole match for Bournemouth, had 10 interceptions. His PAdj interceptions value would be: 10 / 42.5 * 30 = 7.06</em><br><br>There was the formula. Math doesn't lie. In the Wyscout Advance Search, both the interceptions p90 and padj interceptions were listed. Then it was just a matter of doing the math.</p>

<p style="font-size: 16px;">
  <strong>(Interception p90)</strong> / <strong>(Opp Poss Time)</strong> * 30 = <strong>(Padj Interception)</strong>
</p>

<p style="font-size: 16px;">
  Wyscout has Padj versions for both Interception and Sliding Tackle metrics. Some players may have “NA” Tackle numbers and some may have “NA” Interception numbers. Then both can be used. Also, since Wyscout shows a maximum of 2 decimals, there may be a slight difference if the process is applied separately for tackles and interceptions. So I created a function taking into account each case.
</p>


```python
def opp_poss_time(raw_inter=None, padj_inter=None, raw_tackle=None, padj_tackle=None):
    opp_poss_time_from_inter = (raw_inter * 30 / padj_inter) if raw_inter and padj_inter else None
    opp_poss_time_from_tackle = (raw_tackle * 30 / padj_tackle) if raw_tackle and padj_tackle else None

    # Both calculations returned None while None
    if opp_poss_time_from_inter is None and opp_poss_time_from_tackle is None:
        return None

    # If there is only one value, return it, if there are two, return the average
    return opp_poss_time_from_inter if opp_poss_time_from_tackle is None else (
        opp_poss_time_from_tackle if opp_poss_time_from_inter is None else
        (opp_poss_time_from_inter + opp_poss_time_from_tackle) / 2
    )
```

<p style="font-size: 16px;">
  Based on the interception and sliding tackle metrics of English Premier League DMCs for the 2023/2024 season, let's calculate <strong>the opponents' pure possession time when they are on the pitch</strong>
</p>


```python
import pandas as pd
data = pd.read_excel("data.xlsx")

df = data.copy()
df = df[df["Minutes played"] >= 900]
df['opp_poss_time'] = df.apply(
    lambda row: opp_poss_time(row["Interceptions per 90"], row["PAdj Interceptions"], row["Sliding tackles per 90"], row["PAdj Sliding tackles"]), 
    axis=1
)

df.head()
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
      <th>Player</th>
      <th>Team within selected timeframe</th>
      <th>Position</th>
      <th>Minutes played</th>
      <th>Defensive duels per 90</th>
      <th>Defensive duels won, %</th>
      <th>Sliding tackles per 90</th>
      <th>PAdj Sliding tackles</th>
      <th>Interceptions per 90</th>
      <th>PAdj Interceptions</th>
      <th>opp_poss_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bruno Guimarães</td>
      <td>Newcastle United</td>
      <td>DMF, LCMF</td>
      <td>3689</td>
      <td>6.98</td>
      <td>61.89</td>
      <td>0.54</td>
      <td>0.75</td>
      <td>3.46</td>
      <td>4.85</td>
      <td>21.501031</td>
    </tr>
    <tr>
      <th>1</th>
      <td>D. Rice</td>
      <td>Arsenal</td>
      <td>DMF, LCMF</td>
      <td>3601</td>
      <td>4.77</td>
      <td>66.49</td>
      <td>0.47</td>
      <td>0.70</td>
      <td>3.50</td>
      <td>5.14</td>
      <td>20.285436</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Rodri</td>
      <td>Manchester City</td>
      <td>RDMF, DMF, LDMF</td>
      <td>3263</td>
      <td>5.57</td>
      <td>59.90</td>
      <td>0.08</td>
      <td>0.14</td>
      <td>4.03</td>
      <td>6.89</td>
      <td>17.345013</td>
    </tr>
    <tr>
      <th>3</th>
      <td>R. Christie</td>
      <td>Bournemouth</td>
      <td>LDMF, RDMF, LCMF</td>
      <td>3242</td>
      <td>7.77</td>
      <td>58.57</td>
      <td>0.81</td>
      <td>1.02</td>
      <td>4.83</td>
      <td>6.09</td>
      <td>23.808316</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M. Caicedo</td>
      <td>Chelsea</td>
      <td>RDMF, DMF, LDMF</td>
      <td>3216</td>
      <td>6.58</td>
      <td>56.60</td>
      <td>0.31</td>
      <td>0.45</td>
      <td>4.00</td>
      <td>5.89</td>
      <td>20.520091</td>
    </tr>
  </tbody>
</table>
</div>



<p style="font-size: 16px;">
  Let's also change the number of Defensive Duels to PAdj. But before that, let's look at the ranking. Then let's sort by Padj Def Duel.
</p>


```python
selected_columns = ["Player", "Team within selected timeframe", "Position", "Minutes played", "Defensive duels per 90", "Defensive duels won, %", "opp_poss_time"]

df.sort_values("Defensive duels per 90", ascending = False).head(10).filter(selected_columns)
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
      <th>Player</th>
      <th>Team within selected timeframe</th>
      <th>Position</th>
      <th>Minutes played</th>
      <th>Defensive duels per 90</th>
      <th>Defensive duels won, %</th>
      <th>opp_poss_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>26</th>
      <td>N. Domínguez</td>
      <td>Nottingham Forest</td>
      <td>LDMF, RDMF, LCMF</td>
      <td>1651</td>
      <td>10.58</td>
      <td>67.01</td>
      <td>28.226496</td>
    </tr>
    <tr>
      <th>9</th>
      <td>A. Mac Allister</td>
      <td>Liverpool</td>
      <td>DMF, RCMF, LCMF</td>
      <td>2897</td>
      <td>10.13</td>
      <td>57.36</td>
      <td>18.327602</td>
    </tr>
    <tr>
      <th>14</th>
      <td>R. Yates</td>
      <td>Nottingham Forest</td>
      <td>RDMF, RCMF</td>
      <td>2309</td>
      <td>9.86</td>
      <td>64.03</td>
      <td>27.584688</td>
    </tr>
    <tr>
      <th>7</th>
      <td>João Palhinha</td>
      <td>Fulham</td>
      <td>RDMF, LDMF, LCMF</td>
      <td>3022</td>
      <td>9.83</td>
      <td>61.52</td>
      <td>23.755446</td>
    </tr>
    <tr>
      <th>21</th>
      <td>W. Endo</td>
      <td>Liverpool</td>
      <td>DMF</td>
      <td>1950</td>
      <td>9.55</td>
      <td>56.04</td>
      <td>18.167705</td>
    </tr>
    <tr>
      <th>32</th>
      <td>I. Sangaré</td>
      <td>Nottingham Forest</td>
      <td>LDMF, RCMF</td>
      <td>1139</td>
      <td>9.48</td>
      <td>60.00</td>
      <td>28.369565</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Vini Souza</td>
      <td>Sheffield United</td>
      <td>DMF, RCMF</td>
      <td>3019</td>
      <td>8.85</td>
      <td>66.67</td>
      <td>29.744412</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Casemiro</td>
      <td>Manchester United</td>
      <td>DMF, RDMF, RCB</td>
      <td>2226</td>
      <td>8.65</td>
      <td>60.75</td>
      <td>24.553645</td>
    </tr>
    <tr>
      <th>10</th>
      <td>C. Nørgaard</td>
      <td>Brentford</td>
      <td>DMF</td>
      <td>2796</td>
      <td>8.59</td>
      <td>61.05</td>
      <td>24.538352</td>
    </tr>
    <tr>
      <th>30</th>
      <td>S. Lukić</td>
      <td>Fulham</td>
      <td>RDMF, LDMF, RCMF</td>
      <td>1289</td>
      <td>8.59</td>
      <td>52.85</td>
      <td>25.066950</td>
    </tr>
  </tbody>
</table>
</div>




```python
df["PAdj Def Duels"] = df.apply(
    lambda row: row["Defensive duels per 90"] / row["opp_poss_time"] * 30,
    axis = 1
)

selected_columns = ["Player", "Team within selected timeframe", "Position", "PAdj Def Duels", "Defensive duels per 90", "Defensive duels won, %", "opp_poss_time"]

df.sort_values("PAdj Def Duels", ascending = False).head(10).filter(selected_columns)
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
      <th>Player</th>
      <th>Team within selected timeframe</th>
      <th>Position</th>
      <th>PAdj Def Duels</th>
      <th>Defensive duels per 90</th>
      <th>Defensive duels won, %</th>
      <th>opp_poss_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>A. Mac Allister</td>
      <td>Liverpool</td>
      <td>DMF, RCMF, LCMF</td>
      <td>16.581547</td>
      <td>10.13</td>
      <td>57.36</td>
      <td>18.327602</td>
    </tr>
    <tr>
      <th>21</th>
      <td>W. Endo</td>
      <td>Liverpool</td>
      <td>DMF</td>
      <td>15.769741</td>
      <td>9.55</td>
      <td>56.04</td>
      <td>18.167705</td>
    </tr>
    <tr>
      <th>7</th>
      <td>João Palhinha</td>
      <td>Fulham</td>
      <td>RDMF, LDMF, LCMF</td>
      <td>12.413996</td>
      <td>9.83</td>
      <td>61.52</td>
      <td>23.755446</td>
    </tr>
    <tr>
      <th>33</th>
      <td>S. Amrabat</td>
      <td>Manchester United</td>
      <td>LDMF, RCMF, DMF</td>
      <td>11.544308</td>
      <td>8.58</td>
      <td>53.40</td>
      <td>22.296703</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Y. Bissouma</td>
      <td>Tottenham Hotspur</td>
      <td>LDMF, DMF, RDMF</td>
      <td>11.305667</td>
      <td>6.64</td>
      <td>61.31</td>
      <td>17.619482</td>
    </tr>
    <tr>
      <th>26</th>
      <td>N. Domínguez</td>
      <td>Nottingham Forest</td>
      <td>LDMF, RDMF, LCMF</td>
      <td>11.244754</td>
      <td>10.58</td>
      <td>67.01</td>
      <td>28.226496</td>
    </tr>
    <tr>
      <th>31</th>
      <td>R. Bentancur</td>
      <td>Tottenham Hotspur</td>
      <td>LDMF, DMF, RDMF</td>
      <td>11.176129</td>
      <td>6.28</td>
      <td>61.25</td>
      <td>16.857357</td>
    </tr>
    <tr>
      <th>14</th>
      <td>R. Yates</td>
      <td>Nottingham Forest</td>
      <td>RDMF, RCMF</td>
      <td>10.723340</td>
      <td>9.86</td>
      <td>64.03</td>
      <td>27.584688</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Casemiro</td>
      <td>Manchester United</td>
      <td>DMF, RDMF, RCB</td>
      <td>10.568696</td>
      <td>8.65</td>
      <td>60.75</td>
      <td>24.553645</td>
    </tr>
    <tr>
      <th>10</th>
      <td>C. Nørgaard</td>
      <td>Brentford</td>
      <td>DMF</td>
      <td>10.501928</td>
      <td>8.59</td>
      <td>61.05</td>
      <td>24.538352</td>
    </tr>
  </tbody>
</table>
</div>


