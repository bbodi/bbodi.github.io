- ha egy SLSB-nek van egy SFSB fieldje, akkor
miután az SLSB metódushívása véget ér, az visszatér a Poolba, de az
SFSB még tranzakcióban marad. EKkor ha a Poolból
kiveszi egy másik szál a SLSB-t, akkor annak a fieldje
még mindig a tranzakcióban levő SFSB-re mutat, és amikor
ezen a beanen akar majd metódust hívni, AccessTimeoutException lesz.

Mi van, ha szabályosan eltávolítottuk a Stateful-t @Remove-val?

- Írni a veszélyéről: a hívó félnek tudnia kell a hívott viselkedéséről.
- A hívott típusa megváltoztatható, anélkül, hogy a hívó tudna erről.

![Alt text](http://i.imgur.com/4G878Az.png)
![Alt text](http://i.imgur.com/92pHRDR.png)
![Alt text](http://i.imgur.com/uKnlvHr.png)
![Alt text](http://i.imgur.com/92pHRDR.png)
![Alt text](http://i.imgur.com/Igs9B32.png)
