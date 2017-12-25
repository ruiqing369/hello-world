breed [hunters hunter]
breed [livestock_owners livestock_owner]
breed [reindeer_herders reindeer_herder]
breed [wolves wolf]
breed [livestock_ownercors livestock_ownercor]
breed [reindeer_herdercors reindeer_herdercor]
breed [environmentalists environmentalist]
breed [urbanites urbanite]
breed [urbanitecors urbanitecor]



turtles-own [opinion eps opinion-list extremist?]
globals [current_min_eps current_max_eps current_alpha current_beta]

;;entry_exit_rate = 0
;;min_eps = 0
;;max_eps = 0.5
;;alpha = 2
;;beta = 4




to setup
  clear-all
  ask patches
  [set pcolor green - 4]
  create-hunters pop_hunters
  [set shape "triangle"
    set color white
    setxy random-xcor random-ycor
   ]
  create-livestock_owners pop_livestock_owners
  [set shape "triangle 2"
    set color white
    set xcor random 20 - 10
    set ycor random 20 - 40
  ]
  create-reindeer_herders pop_reindeer_herders
  [set shape "circle"
    set color white
    set xcor random 20
    set ycor random 20 + 30
  ]
  create-wolves pop_wolves
  [set shape "wolf"
   set color brown - 1
   set size size + 1
   setxy random-xcor random-ycor
    ]
  create-environmentalists pop_environmentalists
  [set shape "square"
    set color white
    setxy random-xcor random-ycor
  ]
  create-urbanites pop_urbanites
  [set shape "star"
    set color white
    set xcor random 20 + 30
    set ycor random 20 - 10
  ]
  ask turtles
  [set size size + 1
  ]
  create-livestock_ownercors 1
  [set color black
    set xcor 0
    set ycor -30
    set size 0
  ]
  create-reindeer_herdercors 1
  [set color black
    set xcor 10
    set ycor 40
    set size 0
  ]
  create-urbanitecors 1
  [set color black
    set xcor 40
    set ycor 0
    set size 0
  ]
  set_extrimest
  new_confidence_bounds
  set_opinion
  reset-ticks
end

to set_extrimest
  ask (turtle-set environmentalists)
    [set extremist? opinion >= (1 - extremism_range)]
  ask (turtle-set hunters)
    [set extremist? opinion <= (extremism_range)]
  ask (turtle-set urbanites livestock_owners reindeer_herders)
    [set extremist? false]
end

to new_confidence_bounds
  set current_min_eps 0  ;; min_eps is set as 0
  set current_max_eps 0.5  ;; max_eps is set as 0.5
  set current_alpha 2  ;; alpha is set as 2
  set current_beta 4  ;; beta is set as 4
  ask turtles [
    let x random-gamma 2 1
    set eps ( x / ( x + random-gamma 4 1) ) ;; set eps a random number from distribution Beta(alpha,beta) (between 0 and 1)
    set eps 0 + (eps * (0.5 - 0)) ;; scale and shift eps to lie between min_eps and max_eps
    ]
end

to set_opinion
  ask environmentalists
    [set opinion random-normal opinion_environmentalists 0.25]
  ask urbanites
    [set opinion random-normal opinion_urbanites 0.25]
  ask hunters
    [set opinion random-normal opinion_hunters 0.25]
  ask livestock_owners
    [set opinion random-normal opinion_livestock_owners 0.25]
  ask reindeer_herders
    [set opinion random-normal opinion_reindeer_herders 0.25]
end



to go
  ;;if count wolves >= 1500
    ;;[stop]
  wolves_regenerate
  wolves_die
  wolves_move
  wolves_hunters_interaction
  wolves_livestock_owners_interaction
  wolves_reindeer_herders_interaction
  wolves_environmentalists_interaction
  wolves_urbanites_interaction
  hunters_move
  livestock_owners_move
  reindeer_herders_move
  environmentalists_move
  urbanites_move
  ask turtles [opinion_update]
  ask turtles [ entry-exit ]
  ask (turtle-set hunters livestock_owners reindeer_herders environmentalists urbanites)
    [if opinion <= 0
       [set opinion 0]
     set-attitude-color
    ]
  tick
end

to wolves_regenerate
  ask wolves
   [if random 100 < wolves_birth_rate
     [hatch 1
       [set heading random 360]
     ]
   ]
end

to wolves_die
  ask wolves
    [if random 100 < wolves_death_rate
      [die]
      let wolfpopulation count wolves
     ifelse wolfpopulation > carrying_capacity
     [let chance-to-die (wolfpopulation - carrying_capacity) / wolfpopulation
     if random-float 1.0 < chance-to-die
      [ die ]
     ]
      [stop]
    ]
end

to wolves_move
  ask wolves
  [forward random 5
  ]
end

to wolves_hunters_interaction
  ask hunters
  [
    let W one-of wolves in-radius wolves-hunters-encounter
    if W != nobody
        [ifelse licensed_hunting != true
           [set opinion (opinion + hunters_wolves_opinion_change)
            if opinion < 0.5
                [if random 100 < illegal_hunting_rate
                 [ask W [die]
                 ]
                ]
            ]
            [if random 100 < licensed_hunting_rate
                  [ask W [die]
                   set opinion (opinion - hunters_wolves_opinion_change)
                  ]
             ]
         if ecotourism = true
            [set opinion opinion + 0.02]
         if opinion >= 0.6
           [set opinion 0.6]
        ]
  ]
end

to wolves_livestock_owners_interaction
  ask livestock_owners
  [
    let W one-of wolves in-radius wolves-livestock_owners-encounter
    if W != nobody
       [ifelse fencing != true
             [set opinion (opinion + livestock_owners_wolves_opinion_change)]
             [if random 2 < 1
                 [set opinion (opinion - livestock_owners_wolves_opinion_change)]
             ]
        if opinion >= 0.6
             [set opinion 0.6]
        if ecotourism = true
             [set opinion opinion + 0.03]
        ]
  ]
end

to wolves_reindeer_herders_interaction
  ask reindeer_herders
  [
    let W one-of wolves in-radius wolves-reindeer_herders-encounter
    if W != nobody
    [set opinion (opinion + reindeer_herders_wolves_opinion_change)
     if ecotourism = true
          [set opinion opinion + 0.03]
     if opinion >= 0.6
         [set opinion 0.6]
    ]
  ]
end

to wolves_environmentalists_interaction
  ask environmentalists
  [
    let W one-of wolves in-radius wolves-environmentalists-encounter
    if W != nobody
    [
      ifelse count wolves >= carrying_capacity
         [set opinion opinion]
         [set opinion (opinion + environmentalists_wolves_opinion_change)]
      if ecotourism = true
         [set opinion opinion + 0.07]
      if opinion >= 1
         [set opinion 1]
    ]
  ]
end

to wolves_urbanites_interaction
  ask urbanites
  [
    let W one-of wolves in-radius wolves-urbanites-encounter
    if W != nobody
    [
      if count wolves > carrying_capacity
        [set opinion opinion]
      if ecotourism = true
        [set opinion opinion + 0.07]
    ]
    if opinion >= 1
          [set opinion 1]
  ]
end

to hunters_move
  ask hunters
  [forward random 3
  ]
end

to livestock_owners_move
  ask livestock_owners
  [
    if (xcor < -10)
    [set heading towards livestock_ownercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites)
        ]
    if (xcor > 10)
    [set heading towards livestock_ownercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites)
        ]
    if (ycor < -40)
    [set heading towards livestock_ownercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites)
        ]
    if (ycor > -20)
    [set heading towards livestock_ownercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites)
        ]
    forward random 2]
end

to reindeer_herders_move
  ask reindeer_herders
  [
    if (xcor < 0)
    [set heading towards reindeer_herdercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 1)
        ]
    if (xcor > 20)
    [set heading towards reindeer_herdercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 1)
        ]
    if (ycor < 30)
    [set heading towards reindeer_herdercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 1)
        ]
    if (ycor > 50)
      [set heading towards reindeer_herdercor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 1)
        ]
    forward random 2]
end

to environmentalists_move
  ask environmentalists
  [forward random 2
  ]
end

to urbanites_move
  ask urbanites
  [
    if (xcor < 30)
    [set heading towards urbanitecor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 2)
        ]
    if (xcor > 50)
    [set heading towards urbanitecor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 2)
        ]
    if (ycor < -10)
    [set heading towards urbanitecor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 2)
        ]
    if (ycor > 10)
      [set heading towards urbanitecor (pop_hunters + pop_livestock_owners + pop_reindeer_herders + pop_wolves + pop_environmentalists + pop_urbanites + 2)
        ]
    forward random 2]
end

to opinion_update
  let stakeholder one-of (turtle-set hunters livestock_owners reindeer_herders environmentalists urbanites)
  ask stakeholder
  ;; detect if in extremism zone
  [;;set extremist? (0.5 - abs(opinion - 0.5) < extremism_range)
  ;; change of opinion
  let partner one-of (turtle-set hunters livestock_owners reindeer_herders environmentalists urbanites) in-radius communication_range
  if ((partner != nobody) and (partner != self))
   [if (abs (opinion - [opinion] of partner) < eps ) and (not extremist?) [
     set opinion (opinion + [opinion] of partner) / 2
   ]
   ]
  ]
end

to entry-exit
  ;; randomly reset opinion with probability entry_exit_rate
  if (random-float 1 < 0) [set opinion new-opinion] ;;entry_exit_rate is set as 0
end

to set-attitude-color
  ;;set agent's color according to their attitudes
  if opinion >= 0.5
    [ set color green + 4 - ( opinion * 5) ]
  if opinion < 0.5
    [ set color red - ( opinion * 5 ) ]
end

to-report new-opinion
  report random-float 1
end

to-report aggregate_opinion
  let h (mean [opinion] of hunters)
  let f (mean [opinion] of livestock_owners)
  let S (mean [opinion] of reindeer_herders)
  let en (mean [opinion] of environmentalists)
  let u (mean [opinion] of urbanites)
  report ( h + f + S + en + u ) / 5
end
