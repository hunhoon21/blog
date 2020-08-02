<div class="guide-section-description">
      <h6 class="guide-section-title">문제 설명</h6>
      <div class="markdown solarized-dark"><h2>셔틀버스</h2>

<p>카카오에서는 무료 셔틀버스를 운행하기 때문에 판교역에서 편하게 사무실로 올 수 있다. 카카오의 직원은 서로를 '크루'라고 부르는데, 아침마다 많은 크루들이 이 셔틀을 이용하여 출근한다.</p>

<p>이 문제에서는 편의를 위해 셔틀은 다음과 같은 규칙으로 운행한다고 가정하자.</p>

<ul>
<li>셔틀은 <code>09:00</code>부터 총 <code>n</code>회 <code>t</code>분 간격으로 역에 도착하며, 하나의 셔틀에는 최대 <code>m</code>명의 승객이 탈 수 있다.</li>
<li>셔틀은 도착했을 때 도착한 순간에 대기열에 선 크루까지 포함해서 대기 순서대로 태우고 바로 출발한다. 예를 들어 <code>09:00</code>에 도착한 셔틀은 자리가 있다면 <code>09:00</code>에 줄을 선 크루도 탈 수 있다.</li>
</ul>

<p>일찍 나와서 셔틀을 기다리는 것이 귀찮았던 콘은, 일주일간의 집요한 관찰 끝에 어떤 크루가 몇 시에 셔틀 대기열에 도착하는지 알아냈다. 콘이 셔틀을 타고 사무실로 갈 수 있는 도착 시각 중 제일 늦은 시각을 구하여라.</p>

<p>단, 콘은 게으르기 때문에 같은 시각에 도착한 크루 중 대기열에서 제일 뒤에 선다. 또한, 모든 크루는 잠을 자야 하므로 <code>23:59</code>에 집에 돌아간다. 따라서 어떤 크루도 다음날 셔틀을 타는 일은 없다.</p>

<h3>입력 형식</h3>

<p>셔틀 운행 횟수 <code>n</code>, 셔틀 운행 간격 <code>t</code>, 한 셔틀에 탈 수 있는 최대 크루 수 <code>m</code>, 크루가 대기열에 도착하는 시각을 모은 배열 <code>timetable</code>이 입력으로 주어진다.</p>

<ul>
<li>0 ＜ <code>n</code> ≦ 10</li>
<li>0 ＜ <code>t</code> ≦ 60</li>
<li>0 ＜ <code>m</code> ≦ 45</li>
<li><code>timetable</code>은 최소 길이 1이고 최대 길이 2000인 배열로, 하루 동안 크루가 대기열에 도착하는 시각이 <code>HH:MM</code> 형식으로 이루어져 있다.</li>
<li>크루의 도착 시각 <code>HH:MM</code>은 <code>00:01</code>에서 <code>23:59</code> 사이이다.</li>
</ul>

<h3>출력 형식</h3>

<p>콘이 무사히 셔틀을 타고 사무실로 갈 수 있는 제일 늦은 도착 시각을 출력한다. 도착 시각은 <code>HH:MM</code> 형식이며, <code>00:00</code>에서 <code>23:59</code> 사이의 값이 될 수 있다.</p>

<h3>입출력 예제</h3>
<table class="table">
        <thead><tr>
<th>n</th>
<th>t</th>
<th>m</th>
<th>timetable</th>
<th>answer</th>
</tr>
</thead>
        <tbody><tr>
<td>1</td>
<td>1</td>
<td>5</td>
<td>[<q>08:00</q>, <q>08:01</q>, <q>08:02</q>, <q>08:03</q>]</td>
<td><q>09:00</q></td>
</tr>
<tr>
<td>2</td>
<td>10</td>
<td>2</td>
<td>[<q>09:10</q>, <q>09:09</q>, <q>08:00</q>]</td>
<td><q>09:09</q></td>
</tr>
<tr>
<td>2</td>
<td>1</td>
<td>2</td>
<td>[<q>09:00</q>, <q>09:00</q>, <q>09:00</q>, <q>09:00</q>]</td>
<td><q>08:59</q></td>
</tr>
<tr>
<td>1</td>
<td>1</td>
<td>5</td>
<td>[<q>00:01</q>, <q>00:01</q>, <q>00:01</q>, <q>00:01</q>, <q>00:01</q>]</td>
<td><q>00:00</q></td>
</tr>
<tr>
<td>1</td>
<td>1</td>
<td>1</td>
<td>[<q>23:59</q>]</td>
<td><q>09:00</q></td>
</tr>
<tr>
<td>10</td>
<td>60</td>
<td>45</td>
<td>[<q>23:59</q>,<q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>, <q>23:59</q>]</td>
<td><q>18:00</q></td>
</tr>
</tbody>
      </table>
<p><a href="http://tech.kakao.com/2017/09/27/kakao-blind-recruitment-round-1/" target="_blank" rel="noopener">해설 보러가기</a></p>
</div>
    </div>


## 나의 풀이
* 마지막 셔틀이 가득 찼는지 확인하는 것이 포인트
    * 마지막 셔틀이 다 찼으면, 마지막에 탄 사람 직전에 와야함
    * 마지막 셔틀이 다 안 찼으면, 여유롭게 막차 시간에 오면 됨
* 탈 수 있는 크루를 셔틀에 태우기 위해(pop 하기 위해) 자료구조 deque 사용

마지막 셔틀을 확인하기 위해 아래와 같은 순서로 문제를 해결합니다.
1. 정렬된 crew의 timetable 만들기 (`[(hour, min), (hour, min), ...]`)
2. shuttle의 timetable 만들기 (`[(hour, min), (hour, min), ...]`)
3. shuttle 순서에 따라 crew 태우기
  * 마지막 shuttle의 경우 탄 crew 인원을 셉니다.
4. 마지막 셔틀 검사하기
  * 가득 찬 경우 마지막에 탄 크루 직전에 와야함
  * 안 찬 경우 널널하게 막차 시간에 와도됨

## 코드 풀이

### 1. 정렬된 crew의 timetable 만들기
`[(hour, min), (hour, min), ...]` 와 같은 구조로 crew의 timetable을 만들고 정렬합니다.
```python
from collections import deque


def convert_timetable_with_tuple(timetable):
    return [
        (int(time[:2]), int(time[-2:])) for time in timetable
    ]


def sort_timetable(timetable):
    timetable.sort(key=lambda time: time[1])
    timetable.sort(key=lambda time: time[0])
    return timetable


crew_timetable = convert_timetable_with_tuple(crew_timetable)
crew_timetable = sort_timetable(crew_timetable)
crew_timetable = deque(crew_timetable)
```

### 2. shuttle의 timetable 만들기
`[(hour, min), (hour, min), ...]` 와 같은 구조로 crew의 timetable을 만듭니다.
```python
def initialize_shuttle_timetable(n, t):
    shuttle_timetable = [(9, 0)]
    for i in range(n - 1):
        before_hour, before_min = shuttle_timetable[-1]
        after_hour, after_min = before_hour, before_min + t
        if after_min >= 60:
            after_hour, after_min = before_hour + 1, after_min - 60
        shuttle_timetable.append((after_hour, after_min))
    return shuttle_timetable
shuttle_timetable = initialize_shuttle_timetable(n, t)
```

### 3. shuttle 순서에 따라 crew 태우기
남은 크루가 있고, 셔틀에 탈 수 있으면 태웁니다. 막차의 경우엔 탑승한 인원을 셉니다.
```python
def can_take_shuttle(shuttle_time, crew_time):
    # 셔틀 10시 대, 크루 9시 대
    if crew_time[0] < shuttle_time[0]:
        return True
    # 셔틀 10시 10분, 크루 10시 10분 전
    elif crew_time[0] == shuttle_time[0] and crew_time[1] <= shuttle_time[1]:
        return True
    return False


crew_on_last_shuttle = 0
for i, shuttle_time in enumerate(shuttle_timetable):
    for _ in range(m):
        # 남은 크루가 있고, 셔틀에 탈 수 있는 경우
        if (
            crew_timetable and
            can_take_shuttle(shuttle_time, crew_timetable[0])
        ):
            the_last_crew_time = crew_timetable.popleft()
            # 막차의 경우 탑승 인원 세기
            if i == n - 1:
                crew_on_last_shuttle += 1
        else:
            break
```