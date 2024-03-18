#Befudged UP - Reverse challenge from WolvCTF 24
I played WolvCTF with the team [S.h.i.c.h.i.b.u.k.a.i](https://ctftime.org/team/274059) and there was an interesting reverse engineering challenge where there was an implementation of brainfuck like language but in Python.

We are given 3 files:
* [befunge.py](https://gist.github.com/CyberGhost13337/260a186f7bc9f8bc3eb8722b29e655cd#file-befunge-py)
* [runner.py](https://gist.github.com/CyberGhost13337/260a186f7bc9f8bc3eb8722b29e655cd#file-runner-py)
* [prog.befunge](https://gist.github.com/CyberGhost13337/260a186f7bc9f8bc3eb8722b29e655cd#file-prog-befunge)

Obviously we need to regenerate flag from the **prog.befunge**. Instead of reverse engineering what the program do, I just think the output lenght since if program find a correct "char" it needs to do more work. As a example you can run following to see the difference. We know flag should start as wctf{, so when I give "w" and "a" as input latter should have less output.

```bash
➜ python runner.py |wc
a
   6730   17690   57091
➜ python runner.py |wc
w
  12546   33463  105732
```
As you can see, when "a" is input 6730 lines is generated while when the "w" is input 12546 lines is generated. To sure about that I tried to give "wc" and "wa" as input too, to check second characters.
```bash
➜  python runner.py |wc
wa
  12536   33288  105642
➜  python runner.py |wc
wc
  18602   49630  156509
```
With this fact I wrote a script that check every printable character then get the one has huge different with the previous as correct character. I use threading here since this takes long time to finish.

Here is the solution script
[solver.py](https://gist.github.com/CyberGhost13337/260a186f7bc9f8bc3eb8722b29e655cd#file-solve-py)

and flag was `wctf{pr30ccup13d_w1+h_wh3+h3r_0r_n0t_1_c0uld}`