# NexoJobs - Skript Integration Guide

### General Job Information

| Placeholder | Description | Example Output |
|------------|-------------|----------------|
| `%nexojobs_job%` | Current active job name | `MINER` |
| `%nexojobs_job_id%` | Current active job ID | `miner` |
| `%nexojobs_active_count%` | Number of active jobs | `2` |
| `%nexojobs_job_limit%` | Maximum jobs allowed | `5` or `Unlimited` |
| `%nexojobs_has_job%` | Has any active job | `true` or `false` |
| `%nexojobs_status%` | Current job status | `ACTIVE` |

### Current Active Job Stats

| Placeholder | Description | Example Output |
|------------|-------------|----------------|
| `%nexojobs_level%` | Current job level | `25` |
| `%nexojobs_exp%` | Current job EXP | `1500` |
| `%nexojobs_exp_required%` | EXP needed for level | `2000` |
| `%nexojobs_max_level%` | Maximum level | `50` |
| `%nexojobs_percentage%` | Progress percentage | `75.0` |
| `%nexojobs_exp_remaining%` | EXP to next level | `500` |
| `%nexojobs_progress_bar%` | Visual progress bar | `▰▰▰▰▰▰▰▱▱▱` |

### Specific Job Stats
Replace `<jobid>` with: `miner`, `farmer`, `hunter`, `lumberjack`, `enchanter`, `alchemist`, `blacksmith`, `digger`, `chef`, `murderer`

| Placeholder | Description | Example |
|------------|-------------|---------|
| `%nexojobs_<jobid>_level%` | Job level | `%nexojobs_miner_level%` → `30` |
| `%nexojobs_<jobid>_exp%` | Job EXP | `%nexojobs_farmer_exp%` → `5000` |
| `%nexojobs_<jobid>_required%` | EXP required | `%nexojobs_hunter_required%` → `8000` |
| `%nexojobs_<jobid>_percentage%` | Progress % | `%nexojobs_miner_percentage%` → `62.5` |
| `%nexojobs_<jobid>_max_level%` | Max level | `%nexojobs_farmer_max_level%` → `50` |
| `%nexojobs_<jobid>_status%` | Job status | `%nexojobs_miner_status%` → `ACTIVE` |
| `%nexojobs_<jobid>_is_active%` | Is active | `%nexojobs_miner_is_active%` → `true` |
| `%nexojobs_<jobid>_has_started%` | Has progress | `%nexojobs_farmer_has_started%` → `true` |

## Common Use Cases

### 1. Level Requirement Check
```skript
command /vipmine:
    trigger:
        set {_level} to placeholder "nexojobs_miner_level" from player parsed as integer
        if {_level} >= 20:
            teleport player to location(100, 64, 100, world "world")
            send "&aTeleported to VIP Mine!" to player
        else:
            send "&cYou need Miner level 20+ to access VIP Mine!" to player
```

### 2. Multiple Job Requirement
```skript
command /mastershop:
    trigger:
        set {_miner} to placeholder "nexojobs_miner_level" from player parsed as integer
        set {_blacksmith} to placeholder "nexojobs_blacksmith_level" from player parsed as integer
        
        if {_miner} >= 30:
            if {_blacksmith} >= 30:
                send "&aWelcome to Master Shop!" to player
                # Open shop GUI
            else:
                send "&cYou need Blacksmith level 30+ too!" to player
        else:
            send "&cYou need Miner level 30+ and Blacksmith level 30+!" to player
```

### 3. Dynamic Rewards
```skript
on kill of zombie:
    set {_level} to placeholder "nexojobs_hunter_level" from player parsed as integer
    set {_isActive} to placeholder "nexojobs_hunter_is_active" from player
    
    if {_isActive} is "true":
        if {_level} >= 40:
            chance of 10%:
                drop 1 diamond named "&c&lHunter's Trophy"
                send "&e&l+BONUS! &7Hunter level 40+ bonus drop!" to player
```

### 4. Scoreboard Integration
```skript
every 5 seconds:
    loop all players:
        set {_hasJob} to placeholder "nexojobs_has_job" from loop-player
        if {_hasJob} is "true":
            set {_job} to placeholder "nexojobs_job" from loop-player
            set {_level} to placeholder "nexojobs_level" from loop-player
            set {_pct} to placeholder "nexojobs_percentage" from loop-player
            
            set line 5 of loop-player's scoreboard to "&7Job: %{_job}%"
            set line 6 of loop-player's scoreboard to "&7Level: &e%{_level}%"
            set line 7 of loop-player's scoreboard to "&7Progress: &e%{_pct}%%%"
```

### 5. Unlock System
```skript
command /fly:
    trigger:
        set {_total} to 0
        
        loop "miner", "farmer", "hunter", "lumberjack":
            set {_lvl} to placeholder "nexojobs_%loop-value%_level" from player parsed as integer
            add {_lvl} to {_total}
        
        if {_total} >= 100:
            enable flight for player
            send "&aFly enabled! (Total job levels: %{_total}%)" to player
        else:
            send "&cYou need 100 total job levels to fly! (Current: %{_total}%)" to player
```

### 6. Job-Based Economy
```skript
command /deposit:
    trigger:
        set {_blacksmith} to placeholder "nexojobs_blacksmith_level" from player parsed as integer
        
        # Higher level = lower deposit fee
        if {_blacksmith} >= 40:
            set {_fee} to 0.01 # 1% fee
        else if {_blacksmith} >= 20:
            set {_fee} to 0.03 # 3% fee
        else:
            set {_fee} to 0.05 # 5% fee
        
        send "&aYour deposit fee: %{_fee} * 100%%% (Blacksmith level: %{_blacksmith}%)" to player
```

### 7. Achievement System
```skript
every 1 minute:
    loop all players:
        set {_miner} to placeholder "nexojobs_miner_level" from loop-player parsed as integer
        
        if {_miner} is 50:
            if {achievement.miner.master::%loop-player%} is not set:
                set {achievement.miner.master::%loop-player%} to true
                broadcast "&e&l[ACHIEVEMENT] &r%loop-player% &7became a &b&lMaster Miner!"
                give loop-player 1 diamond block named "&b&lMaster Miner Trophy"
```

### 8. Shop Discounts
```skript
command /buy <text>:
    trigger:
        if arg-1 is "pickaxe":
            set {_price} to 1000
            set {_blacksmith} to placeholder "nexojobs_blacksmith_level" from player parsed as integer
            
            if {_blacksmith} >= 30:
                set {_price} to {_price} * 0.7 # 30% off
            else if {_blacksmith} >= 15:
                set {_price} to {_price} * 0.85 # 15% off
            
            if player's balance >= {_price}:
                remove {_price} from player's balance
                give player 1 diamond pickaxe
                send "&aYou bought a pickaxe for $%{_price}%!" to player
```
