- ha egy SLSB-nek van egy SFSB fieldje, akkor 
miután az SLSB metódushívása véget ér, az visszatér a Poolba, de az
SFSB még tranzakcióban marad. EKkor ha a Poolból
kiveszi egy másik szál a SLSB-t, akkor annak a fieldje
még mindig a tranzakcióban levő SFSB-re mutat, és amikor
ezen a beanen akar majd metódust hívni, AccessTimeoutException lesz.
