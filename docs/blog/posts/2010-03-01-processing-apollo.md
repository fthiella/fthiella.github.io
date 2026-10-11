---
date: 2010-03-01
---
# Processing Apollo

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj7Lu65YUGWZtlz6UOyvEcLYjUFD-R13jweLnoOl-gUUiHfY_gdPNv0uruRVMT_btAHqimRfOM0I_wv8JaRhT5ENxyBnwqnkl-ej6VKwNuMwU0cz7JIvGZR4gT93VHHrswHd9bJPk2PoIU/s200/Apollo_gross.jpg)](<https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj7Lu65YUGWZtlz6UOyvEcLYjUFD-R13jweLnoOl-gUUiHfY_gdPNv0uruRVMT_btAHqimRfOM0I_wv8JaRhT5ENxyBnwqnkl-ej6VKwNuMwU0cz7JIvGZR4gT93VHHrswHd9bJPk2PoIU/s1600-h/Apollo_gross.jpg>)  
Sono sempre rimasto affascinato dai colori degli anni '70, ma non me la sono sentita di [rendere la mia casa un pochino troppo scioccante](<http://www.cartadaparatideglianni70.com/product_info.php?info=p110_Apollo.html>). Non adesso, insomma. Magari più avanti ci penserò.  
  
Per ora, credo di potermi accontentare di decorare allegramente il desktop del mio computer. E quale migliore occasione per sperimentare un po' di Processing?  
  
Per prima cosa, vediamo di ricavare i **cinque colori** utilizzati nella figura, ad esempio in formato RGB, e per questo basta un qualsiasi programma di grafica:  
  

    
    int larghezza_figura = 100;
    int altezza_figura = 100;
    
    size(larghezza_figura*5, altezza_figura);
    smooth();
    noStroke();
    
    pushMatrix();
    fill(110, 50, 22); // colore 1
    rect(0, 0, larghezza_figura, altezza_figura);
    
    translate(larghezza_figura, 0);
    fill(226, 180, 25); // colore 2
    rect(0, 0, larghezza_figura, altezza_figura);
    
    translate(larghezza_figura, 0);
    fill(220, 115, 0); // colore 3
    rect(0, 0, larghezza_figura, altezza_figura);
    
    translate(larghezza_figura, 0);
    fill(197, 75, 2); // colore 4
    rect(0, 0, larghezza_figura, altezza_figura);
    
    translate(larghezza_figura, 0);
    fill(131, 37, 11); // colore 5
    rect(0, 0, larghezza_figura, altezza_figura);
    popMatrix();
    
    
Anche se i colori non sono precisi, va bene lo stesso, ed il risultato è una combinazione molto vintage e molto calda:  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhLFr2X8sRctMezilhexPUPiKgEoaz5oxH0UJr-R2s9CbdSR4Y_gxsyq4ITh6zUJkogx3H07D1xTwcpJGmVBc_wdLgJLFVrBYgoy_y31C2LtNxC2g02Da1AuXN1lr7CXi6NcsGLs4PirU4/s320/figura1.jpg)](<https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhLFr2X8sRctMezilhexPUPiKgEoaz5oxH0UJr-R2s9CbdSR4Y_gxsyq4ITh6zUJkogx3H07D1xTwcpJGmVBc_wdLgJLFVrBYgoy_y31C2LtNxC2g02Da1AuXN1lr7CXi6NcsGLs4PirU4/s1600-h/figura1.jpg>)
Penso che ci saranno molte occasioni in cui potrò riciclarla!  
  
E per le **dimensioni** dei cerchi, che vanno via via a rimpicciolirsi? La scala non è fissa, e un buon risultato si ottiene partendo da una scala di 0,85 moltiplicata di volta in volta per 0,9. Ecco un esempio:  

    
    float dim_figura = 160;
    float scala = 0.85;
    float varia = 0.9;
    
    size(int(dim_figura), int(dim_figura));
    smooth();
    noStroke();
    
    pushMatrix();
    background(110, 50, 22); // colore sfondo
    
    dim_figura = dim_figura*scala;
    
    fill(226, 180, 25); // colore 2
    ellipse(width/2, height/2, dim_figura, dim_figura);
    
    fill(220, 115, 0); // colore 3
    scala = scala*varia;
    
    dim_figura = dim_figura*scala;
    ellipse(width/2, height/2, dim_figura, dim_figura);
    
    fill(197, 75, 2); // colore 4
    scala = scala*varia;
    dim_figura = dim_figura*scala;
    ellipse(width/2, height/2, dim_figura, dim_figura);
    
    fill(131, 37, 11); // colore 5
    scala = scala*varia;
    dim_figura = dim_figura*scala;
    ellipse(width/2, height/2, dim_figura, dim_figura);
    
    popMatrix();
    
  
Le proporzioni ci assomigliano, ma manca ancora qualche cosa:  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg1T4Fv1hIWkoUN5pR890L4eRf4FYg-1AwGPb4rOkAZ13SKR1G28Ndcj1wAIdrgVMJCUnylY9h4jynS7ZsiYvLu7ke0_zFfoMg-gmsKLnHuOXGBFSFwaoUHzYej3CC5Hf0tA0kEDRs1hkE/s1600/figura2.jpg)](<https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg1T4Fv1hIWkoUN5pR890L4eRf4FYg-1AwGPb4rOkAZ13SKR1G28Ndcj1wAIdrgVMJCUnylY9h4jynS7ZsiYvLu7ke0_zFfoMg-gmsKLnHuOXGBFSFwaoUHzYej3CC5Hf0tA0kEDRs1hkE/s1600-h/figura2.jpg>)
Sono certo che si possa fare di meglio, ma per ora il codice finale è il seguente:  

    
    float dim_figura = 160;
    float scala = 0.85;
    float varia = 0.9;
    
    size(int(dim_figura), int(dim_figura));
    smooth();
    noStroke();
    
    pushMatrix();
    translate(width/2,height/2);
    rotate(radians(45));
    background(110, 50, 22); // colore sfondo
    
    fill(226, 180, 25); // colore 2
    
    dim_figura = dim_figura*scala;
    ellipse(0, 0, dim_figura, dim_figura);
    
    fill(220, 115, 0); // colore 3
    scala = scala*varia;
    
    translate((dim_figura-dim_figura*scala)/2, 0);
    dim_figura = dim_figura*scala;
    ellipse(0, 0, dim_figura, dim_figura);
    
    fill(197, 75, 2); // colore 4
    scala = scala*varia;
    
    translate((dim_figura-dim_figura*scala)/2, 0);
    dim_figura = dim_figura*scala;
    ellipse(0, 0, dim_figura, dim_figura);
    
    fill(131, 37, 11); // colore 5
    
    scala = scala*varia;
    translate((dim_figura-dim_figura*scala)/2, 0);
    dim_figura = dim_figura*scala;
    ellipse(0, 0, dim_figura, dim_figura);
    
    translate(-width/2,-height/2);
    popMatrix();
    
  
e il risultato, non sembra niente male!  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgAUFVASVJxdkopSzPU-St0jgEs_jLN8g5O2MDw1h_tbtg3Fj65NBu40YXYX44xmD9KbTmUiavJfqcEt755ZP7HYG98W5yfV2JRngrVP1qA0WxmQomH5PxZ0TdJAaGJi5R18In_Pml7S8g/s1600/figura3.jpg)](<https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgAUFVASVJxdkopSzPU-St0jgEs_jLN8g5O2MDw1h_tbtg3Fj65NBu40YXYX44xmD9KbTmUiavJfqcEt755ZP7HYG98W5yfV2JRngrVP1qA0WxmQomH5PxZ0TdJAaGJi5R18In_Pml7S8g/s1600-h/figura3.jpg>)
