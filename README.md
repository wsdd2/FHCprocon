# FHCprocon
All plugin codes and some things about my FHC server
# The things you may need to know about insane limits for votenuke function
This is the part of limit_1_second_check_expression, and rest of setup you need are below:<br>
limit_1_state: Enabled<br>
limit_1_name: Votenuke<br>
limit_1_evaluation: OnAnyChat<br>
limit_1_first_check: Expression<br>
limit_1_first_check_expression: (Regex.Match(server.Gamemode, @"(Conquest|Chain Link)").Success)<br>
limit_1_second_check: code<br>

# The reason why I quit to run a Battlefield 4 server
DDoS attacks happen almost every night, and i3D.net (I rent server from them) can't stop this situation, it seems that they have never 
taken any effective steps to solve this terrible problem. What's more , i3D.net is the only ISP who provides Aisa servers for Battlefield 
4, which means I have no choice but to rent from them. I'm really tired of fighting against unknown-source attackers. So I decide to quit this.

# But Now I'm back, and I will keep updating the codes
All my codes of proconrulz, insane limits and ADKats serires plugins are open to everyone. Welcome to copy and apply!
If you want to be my server's admin or VIP (reserved slots) , contact qq 351344673 or wechat wsdd_FHC.
Donation Alipay: 351344673@qq.com
:P
