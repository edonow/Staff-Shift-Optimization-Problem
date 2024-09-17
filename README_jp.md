# Staff Shift Optimization Problem

このコードは、スタッフのシフトスケジュールを最適化し、効率的かつ公平な勤務割り当てを実現するためのものです。3 パターンのシフトスケジュールを想定しています。

## シフトスケジュールのパターン

1. **シフトパターンが 1 つの場合**

   - 全スタッフが働く時間帯が同じ。

2. **シフトパターンが 2 つの場合**

   - 全スタッフが働く時間帯が朝もしくは昼からの時間帯。

3. **シフトパターンが不規則な場合**
   - 平日は朝と昼の 2 パターン、週末は朝、昼、夜の 3 つのパターン。

## データの読み込みと準備

- スタッフ情報と希望休暇日を含む Excel ファイルを読み込みます。
- 特定期間内の営業日や休業日を計算し、各スタッフの休暇希望も取り込みます。

## 制約条件

### シフトパターンが 1 つの場合

#### 各営業日に 2 名のスタッフが必要:

```python
problem += lpSum(x[s, d] for s in STAFFS) == 2
```

#### 各スタッフは 1 週間に最大 5 日まで勤務:

```python
problem += lpSum(x[s, d] for d in week) <= 5

```

#### 各営業日に少なくとも 1 名のフルタイムスタッフが必要:

```python
problem += lpSum(x[s, d] * S_FULLTIME_OR_NOT[s] for s in STAFFS) >= 1
```

#### スタッフ間の勤務日数差が 1 日以内:

```python
problem += lpSum(x[staff_i, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) - lpSum(x[staff_j, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) <= 1
problem += lpSum(x[staff_i, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) - lpSum(x[staff_j, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) <= 1

```

#### 各各スタッフは最低 1 シフトを担当:

```python
problem += lpSum(x[s, d] for d in WORKING_DAYS_EXCLUDING_CLOSED) >= 1
```

#### 各営業日のスタッフレベルに大きな差がないように調整:

```python
problem += level_sum >= min_value + max_value
problem += level_sum <= max_value + max_value
```

#### スタッフの休暇希望を尊重:

```python
problem += x[staff, day] == 0
```

---

### シフトパターンが 2 つおよび不規則な場合

#### 各営業日の各シフトに 1 名のスタッフが必要:

```python
for d in WORKING_DAYS_EXCLUDING_CLOSED:
    for shift in SHIFT_TYPE:
        problem += lpSum(x[s, d, shift] for s in STAFFS) == 1
```

#### 各スタッフは 1 日に 1 つのシフトしか担当しない:

```python
for s in STAFFS:
    for d in WORKING_DAYS_EXCLUDING_CLOSED:
        problem += lpSum(x[s, d, shift] for shift in SHIFT_TYPE) <= 1
```

#### スタッフが「夜のシフト」を担当した翌日に「朝のシフト」を担当しない:

```python
for d in WORKING_DAYS_EXCLUDING_CLOSED:
    next_day = d + 1
    if next_day in WORKING_DAYS_EXCLUDING_CLOSED:
        for s in STAFFS:
            problem += x[s, d, "E"] + x[s, next_day, "M"] <= 1
```

#### シフトタイプが動的に取得:

シフトタイプが固定された配列（例: ["M", "E"]や["M", "E", "N"]）ではなく、get_shift_types(d)関数を使って動的に取得されるようになっています。これにより、日ごとに異なるシフト構成が可能です。

```python
for shift in get_shift_types(d):
```

#### 夜勤から朝勤へのシフト禁止制約:

```python
if "N" in get_shift_types(d) and "M" in get_shift_types(next_day):
    problem += x[s, d, "N"] + x[s, next_day, "M"] <= 1
```

## 使用方法

1. Excel ファイルを指定して、スタッフ情報と休暇希望を読み込みます。
2. 営業日や休業日、シフトパターンに基づいて制約を設定します。
3. pulp ライブラリを使用して、スタッフの最適なシフト割り当てを計算します。
4. 計算結果を出力し、シフトスケジュールを確認します。
