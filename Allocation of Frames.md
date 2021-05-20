## 0. 들어가면서
* 지난 시간에 page replacement와 관련된 여러가지 알고리즘에 대해 공부했다.
* 이번 시간에는 그렇다면 "프로세스에게 얼마만큼의 frame을 할당해야 하는가?"에 관련된 내용과 그 외의 여러 이슈에 대해 공부해보려고 한다.

---

## 1. Allocation of Frames
* Virtual memory managemnt는 애초에 physical memory가 virtual memory보다 훨씬 작기 때문에 이와 관련된 이슈를 다루는 데 방점이 있었다.
* 따라서 각 프로세스는 자신이 필요로 하는 "최소한의" page만 사용하는 것이 합리적이라고 볼 수 있고, 이와 관련해서 얼마만큼의 frame을 할당해주는 것이 좋을까에 대해 고민해봐야 한다.
* 한편 frame을 할당한 이후, 프로세스에서 page fault가 일어났을 때 이를 대응하는 방식은 크게 두 가지가 있다.
#### 1) Local Replacement
```
- 이 방법은 Fixed allocation 개념인데, 애초에 프로세스 별로 할당된 frame의 크기가 고정되어 있어서 page fault가 발생하면 프로세스 안에서 자체적으로 해결해야 한다.
- 예를 들어 100개의 frame이 있고, 5개의 process가 존재하면 한 프로세스마다 20개의 page 공간을 할당해주는 것이다.
```
#### 2) Global Replacement
```
- 이 방법은 Priority allocation 개념으로, 프로세스의 우선순위에 따라서, 프로세스의 크기에 따라서 필요로 하는 프레임만큼 동적으로 할당해준다. 
- 그래서 만약 page fault가 발생하면 이전에 배운 방식으로 page fault를 핸들링 할 수도 있지만 다른 프로세스에서 frame을 뺐어와서(?) 사용할 수도 있다.
- 예를 들어 전체 frame의 수가 64개이고, process 1,2의 사이즈가 각각 s1, s2라면 process1에게 64 * (s1/(s1+s2)) 만큼의 프레임을 할당해주는 것이다.
```


## 2. Thrashing
```
- 만약 한 프로세스가 사용할 수 있는 physical memory의 공간이 부족하다고 해보자. 그러면 당연히 page fault의 가능성이 높아지고 이를 처리하는 데 overhead가 커진다.
- Thrashing은 이처럼 프로세스들이 제대로 자신의 일을 못하고 page fault를 처리하는 데 시간을 계속 쓰는 상황을 가리킨다.

- 예를 들어서, 현재 CPU의 Utilization이 낮은 상황이라고 해보자. 그러면 OS는 Utilization을 높이기 위해서 multiprogramming 수준을 높이기 위해 더 많은 프로세스를 불러들인다.
- 그런데 문제는 지금 Utilization이 낮은 이유가 프로세스가 널널해서 그런 것이 아니라, frame은 부족하고 프로세스는 많아서 프로세스들이 page fault를 처리하느라 바빴던 것이다.
- 따라서 이런 상황은 CPU Utilization을 더 낮추게 되고, Thrashing 현상을 심화시킨다.

- 다시 말해서 각 프로세스는 Locality에 대한 페이지를 frame에 매핑해놔야 잘 돌아가게 되는데, 이러한 페이지들을 올릴 frame이 부족하면 안 돌아가는 것이다!
```
![image](https://user-images.githubusercontent.com/61929745/118931406-ee9b3180-b981-11eb-8851-2ae10bed4142.png)

* 그리고 아래는 Page-fault Frequency와 frame Allocation을 어떻게 해야하는가에 대한 그림으로, 내가 허용하는 page fault의 rate를 상한선과 하한선으로 정해둔 것이다.
* 그래서 만약 page fault가 상한선 위로 나타나면 frame을 더 할당하고, 반대 상황에서는 frame의 수를 줄여버린다.   
![image](https://user-images.githubusercontent.com/61929745/118933962-c5c86b80-b984-11eb-8744-5d4f8143d426.png)


---

## 3. Working-Set Model
#### 1. Working set 개념
```
- Working-Set이란 "현재 사용되고 있는 페이지"를 의미한다.
- 그리고 Working-Set Window란 page를 reference하는 횟수를 의미하는데, 예를 들어 10,000번의 instruction을 Working-set Window로 정하면, 현재 시점 t로부터 이전의 10,000개의 instruction에 속하는 녀석들을 관찰하고 Working-set을 정하게 된다.
- 아래 그림을 보면 이해하기 좋을 것 같다.   
```
![image](https://user-images.githubusercontent.com/61929745/118932008-87ca4800-b982-11eb-908c-20c6198ac12a.png)

* 한편, 위 그림을 보면 알 수 있다시피 동일한 프로세스가 진행되는 중에도 time에 따라 active하게 쓰이는 page가 다르다는 것을 알 수 있다.
* 그리고 이러한 working-set의 관점에서 만약 WS의 크기가 10인데, 이 시점에서 프로세스에게 frame을 100개 할당해주면 90개는 의미없는 할당이 된다.

#### 2. Working set model detail
```
- WSS는 Working set size of process P의 약자로, 가장 최근의 working-set window에서 사용되고 있는 page들의 수를 의미한다.
- 이 때, 조금만 생각해보면 WS window의 크기가 클수록 더 많은 instruction이 WS에 포함되고 Locality의 개념이 더 커지게 된다.
- 반면 WS window 크기가 작아지면 locality를 인정하는 instruction들이 적어지게 된다.

- 한편 만약 D = 모든 프로세스의 WSS의 합이라고 정의하면, D > total # of frames in physical memory일 때 thrasing이 발생하게 된다.
- 그래서 위와 같은 상황이 발생하면 프로세스들 중 몇 개를 swap out해서 메모리를 확보해야 한다.
```
* 그렇다면 우리는 매 시점 t마다 working set을 일일이 다시 파악해야 할까? 그러면 overhead가 장난아닐 것이다.
* 따라서 이전에 page replacement algorithm에서 clock algorithm을 이용한 것처럼, "Interval time" + "a reference bit"을 통해 WS를 결정하게 된다.
```
// Keeping Track of the Working Set
1. Window 크기가 10,000이라고 하자.
2. 그리고 Interval timer를 통해 5,000 time unit마다 interrupt를 건다고 하자.
3. interrupt를 걸 때, 페이지들의 reference bit를 다른 메모리 장소에다가 복사해놓고, reference bit을 0으로 바꿔버린다.
4. 그러면 interval이 5,000 time unit이기 때문에 10,000 time unit이 지나면 각 페이지들은 2bit의 history bit를 갖게 된다.
5. 이 때, history bit가 둘 다 0이면? --> 두 번의 interrupt에서 모두 사용되지 않았기 때문에 WS에 포함되지 않는 것이다.

- 다만, interval이 5,000 time unit이기 때문에 완벽한 측정은 아니다.
- 그렇다고 해서 interval을 매우 작게 만들면 그 만큼 history bit를 저장해야 할 메모리 공간이 늘어나고, 오버헤드가 커지는 단점이 있다.
- 즉, 적당하게 Window 크기를 정해야 한다는 뜻이다.
```

---

## 4. Other Issues
* 이제까지 frame allocation을 어떻게 할 것인가, Thrashing은 무엇이고 왜 일어나는가, Working-Set은 무엇인지에 대해 공부했다.
* 그리고 프로세스들의 Working-Set 크기의 합이 현재 사용 가능한 Physical memory의 크기(frame의 수)보다 작을 때, Thrashing이 발생할 수 있음을 알았다.
* 이제부터는 Virtual memory, Page fault 등 지금까지 배운 내용과 관련된 자잘한 몇 가지 이슈에 대해 간략히 알아보고자 한다.
