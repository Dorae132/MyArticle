# Git 踩坑记
>Author:Dorae  
>Date:2017年10月16日09:33:37

----

## git pull
	git pull origin master:master  
&emsp;&emsp;当前所处分支为develop,那么当执行此命令时，git的操作是先在master上执行pull，然后再自动在当前所处分支（develop）执行git merge master命令，所以在merge代码之前，最好切换当前分支为对应分支，避免两个分支都有冲突，从而造成风险。