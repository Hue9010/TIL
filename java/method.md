
Collections.sort(List<T> list)
-------
```
public static <T extends Comparable<? super T>> void sort(List<T> list) {
       list.sort(null);
   }
```
- Collections의 sort를 사용하기 위해서는 implements Comparable<Type> 상속 받고 compareTo(T o)를 구현하여야 한다.
```
  >public int compareTo(T o);
```
구현 방식에 따라 내림차순, 오름차순을 결정 지을 수 있다.
체스게임의 말인 [piece](https://github.com/Hue9010/codesquad-chess-game/blob/master/src/pieces/Piece.java)를 점수에 따른 내림차순 정렬을 위해 구현하였다.(sort를 사용하는 곳은 [board](https://github.com/Hue9010/codesquad-chess-game/blob/master/src/chess/Board.java))
- String의 내부엔 다음과 같이 구현되어 있다.
```
public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```
