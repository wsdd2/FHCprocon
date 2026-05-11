# Insane Limits Vote Nuke

This file contains a ready-to-paste Insane Limits setup for vote nuke.

Compared with ProconRulz, Insane Limits can iterate `team1.players` / `team2.players` and immediately call `plugin.KillPlayer(...)`, so it is a better fit for nuking every eligible player on the leading team.

## Limit Setup

Create one new limit in Insane Limits:

- `name`: `Vote Nuke`
- `state`: `Enabled` while live, `Virtual` while testing
- `evaluation`: `OnAnyChat`
- `action`: `None`
- `first_check`: `Code`
- `second_check`: `Disabled`

Paste the code below into `first_check`.

## Code

```csharp
// FHC Vote Nuke for Insane Limits
// Commands:
//   !votenuke - losing team starts a nuke vote when ticket gap is high enough
//   !yes      - losing team votes yes
//
// Rules:
//   - server.PlayerCount must be > 10
//   - ticket gap must be over 50% of the balance ticket gap
//   - only the losing team can start/vote
//   - yes votes must be at least half of losing team size, rounded up
//   - when passed, nuke leading team players with KDR round > 1 and kills round > 5

const double BalanceTicketGap = 75.0;
double nukeTicketGap = Math.Ceiling(BalanceTicketGap * 0.5);

string chat = (player.LastChat ?? "").Trim().ToLowerInvariant();
if (chat != "!votenuke" && chat != "!yes")
{
    return false;
}

int teamId = player.TeamId;
if (teamId != 1 && teamId != 2)
{
    plugin.SendPlayerMessage(player.Name, "Vote nuke is only available for active teams.");
    return false;
}

double t1 = server.RemainTickets(1);
double t2 = server.RemainTickets(2);
int leadingTeam = 0;
int losingTeam = 0;
double ticketGap = Math.Abs(t1 - t2);

if (t1 > t2)
{
    leadingTeam = 1;
    losingTeam = 2;
}
else if (t2 > t1)
{
    leadingTeam = 2;
    losingTeam = 1;
}

if (chat == "!votenuke")
{
    if (server.PlayerCount <= 10)
    {
        plugin.SendPlayerMessage(player.Name, "Vote nuke requires more than 10 players.");
        return false;
    }

    if (leadingTeam == 0 || ticketGap <= nukeTicketGap)
    {
        plugin.SendPlayerMessage(player.Name, "Ticket gap is not high enough for vote nuke.");
        return false;
    }

    if (teamId != losingTeam)
    {
        plugin.SendPlayerMessage(player.Name, "Only the losing team can start vote nuke.");
        return false;
    }

    if (server.RoundData.getBool("nuke_active"))
    {
        plugin.SendPlayerMessage(player.Name, "Vote nuke is already active. Losing team can type !yes.");
        return false;
    }

    int nukeId = server.RoundData.getInt("nuke_id") + 1;
    int losingCount = 0;
    if (losingTeam == 1)
    {
        losingCount = team1.players.Count;
    }
    else if (losingTeam == 2)
    {
        losingCount = team2.players.Count;
    }
    int needed = (losingCount + 1) / 2;

    server.RoundData.setInt("nuke_id", nukeId);
    server.RoundData.setBool("nuke_active", true);
    server.RoundData.setInt("nuke_losing_team", losingTeam);
    server.RoundData.setInt("nuke_leading_team", leadingTeam);
    server.RoundData.setInt("nuke_votes", 0);
    server.RoundData.setInt("nuke_needed", needed);

    plugin.SendGlobalMessage("[AUTOADMIN] Vote nuke started by " + player.Name + ". Losing team type !yes. Need " + needed + " yes votes.");
    plugin.SendGlobalYell("Vote nuke started. Losing team type !yes.", 10);
    return false;
}

if (chat == "!yes")
{
    if (!server.RoundData.getBool("nuke_active"))
    {
        plugin.SendPlayerMessage(player.Name, "No active vote nuke.");
        return false;
    }

    int losingTeamStored = server.RoundData.getInt("nuke_losing_team");
    int leadingTeamStored = server.RoundData.getInt("nuke_leading_team");

    if (teamId != losingTeamStored)
    {
        plugin.SendPlayerMessage(player.Name, "Only the losing team can vote yes.");
        return false;
    }

    int nukeId = server.RoundData.getInt("nuke_id");
    string voteKey = "nuke_voted_" + nukeId + "_" + player.Name;

    if (server.RoundData.getBool(voteKey))
    {
        plugin.SendPlayerMessage(player.Name, "You already voted yes for vote nuke.");
        return false;
    }

    server.RoundData.setBool(voteKey, true);

    int votes = server.RoundData.getInt("nuke_votes") + 1;
    int needed = server.RoundData.getInt("nuke_needed");
    server.RoundData.setInt("nuke_votes", votes);

    plugin.SendGlobalMessage("[AUTOADMIN] Vote nuke yes: " + votes + "/" + needed + ".");

    if (votes < needed)
    {
        return false;
    }

    server.RoundData.setBool("nuke_active", false);

    plugin.SendGlobalMessage("[AUTOADMIN] Vote nuke passed. Leading team will be nuked now.");
    plugin.SendGlobalYell("Vote nuke passed. Leading team nuke incoming.", 10);

    int killed = 0;

    if (leadingTeamStored == 1)
    {
        foreach (PlayerInfoInterface target in team1.players)
        {
            if (target.KdrRound > 1.0 && target.KillsRound > 5.0)
            {
                plugin.SendPlayerYell(target.Name, "Nuked by vote balance.", 8);
                plugin.KillPlayer(target.Name, 1000);
                killed++;
            }
        }
    }
    else if (leadingTeamStored == 2)
    {
        foreach (PlayerInfoInterface target in team2.players)
        {
            if (target.KdrRound > 1.0 && target.KillsRound > 5.0)
            {
                plugin.SendPlayerYell(target.Name, "Nuked by vote balance.", 8);
                plugin.KillPlayer(target.Name, 1000);
                killed++;
            }
        }
    }

    plugin.SendGlobalMessage("[AUTOADMIN] Vote nuke executed. Nuked " + killed + " leading-team players.");
    return false;
}

return false;
```

## Notes

- Keep `action` as `None`; the code performs all messages and kills itself.
- Test with `state = Virtual` first, then switch to `Enabled`.
- If you want the threshold to match another balance limit, change `BalanceTicketGap`.
- `server.RoundData` is automatically cleared when a new round starts.
