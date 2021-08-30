---
author: "Snippers"
title: "AI Bounding Overwatch"
date: "2018-02-03"
description: "Attempting AI to  fire and maneuver tactics"
tags: ["Arma"]
categories: ["themes", "syntax"]
series: ["Arma Missions"]
ShowToc: true
TocOpen: false
---

Arma 3's default AI typically do not coordinate beyond a group level and have limited coordination inside a group. This leads to sub-optimal performance in firefights and more unrealistic behaviour from AI. As it seems they tend to take foolish actions. This motivated me to attempt to improve AI behaviour with some realistic tactical coordination.

Now to borrow some military theory. A significant change in military tactics was the discovery and adaptation of firearms. Which led to [fire and maneuver](https://en.wikipedia.org/wiki/Fire_and_movement) tactics. These are fairly central to coordinated realistic coordinated behaviour. The general principle is that elements/units receiving fire are less effective and typically become suppressed. The next key idea is that advancing elements can be supported by other elements offer covering fire by engaging any enemy forces, hopefully preventing enemy forces from retaliating whilst receiving fire.

I settled on opting to attempt to implement [Bounding overwatch](https://en.wikipedia.org/wiki/Bounding_overwatch). Bounding overwatch is where an element advances (Bounding Element) and another provides fire support (Overwatching Element).

I carried out two distinct efforts on this. Firstly an earlier one in 2015 focusing on intergroup bounding which one AI group would bound and the other would overwatch. Then later in 2018 I decided to try and get this working inside a group, which half the group doing each task.

# Intergroup Bounding
Take Attempt from Aussie Hold out


```sqf
// Bounding OVerwatch assault!
// [group1, group2, targetPos] execVM "bounding.sqf";
private ["_group1" ,"_group2", "_targetPos", "_centerPos" ,"_direction", "_grpPos1", "_grpPos2","_startPos", "_tempPos", "_boundDist", "_grpDist", "_wp"];
_group1 = _this select 0;//group1;
_group2 = _this select 1;//group2;
_targetPos = _this select 2; //getMarkerPos "aiAttackMarker";

_grpPos1 = [];
if (count (waypoints _group1) > 0) then {
	_grpPos1 =(getWPPos ((waypoints _group1) select (count waypoints _group1)-1));
} else { _grpPos1 = getPos (leader _group1); };
_grpPos2 = [];
if (count (waypoints _group2) > 0) then {
	_grpPos2 = (getWPPos ((waypoints _group2) select (count waypoints _group2)-1));
} else { _grpPos2 = getPos(leader _group2); };

_centerPos = (_grpPos1 vectorAdd _grpPos2) vectorMultiply 0.5;
_direction = [_centerPos,_targetPos] call BIS_fnc_dirTo;

_boundDist = 50;
_grpDist = _grpPos1 distance _grpPos2;


_startPos = _grpPos1;
_tempPos = [0,0,0];
_group1 setVariable ["bounding_buddy",_group2];
_group2 setVariable ["bounding_buddy",_group1];

{ _x disableAI "FSM";} forEach (units _group1);
{ _x disableAI "FSM";} forEach (units _group2);
_group1 allowFleeing 0;
_group2 allowFleeing 0;

//
// bounding to overwatch.
_wp = _group1 addWaypoint [_startPos,0];
_wp setWaypointBehaviour "AWARE";
_wp setWaypointStatements ["true", "(group this) call fn_startOverwatching;"];
_wp setWaypointCompletionRadius 1;
_wp setWaypointType "MOVE";

_wp = _group1 addWaypoint [_startPos,0];
_wp setWaypointCompletionRadius 1;
_wp setWaypointType "MOVE";
_wp setWaypointStatements ["(group this) getVariable ['att_okayToBound',false]","(group this) setVariable ['att_okayToBound',false]; (group this) call fn_startBounding; "];

_wp = _group2 addWaypoint [_grpPos2,0];
_wp setWaypointStatements ["true","(group this) setVariable ['att_okayToBound',false]; (group this) call fn_startBounding; "];
_wp setWaypointCompletionRadius 1;
_wp setWaypointType "MOVE";


/*
Problems: Group leader is unconscious, the group stops moving.
- AI when starting overwatch dont lay down a large amount of fire (unless they seem have a viabletarget?)
- AI planning,cover routines take over.
*/


while {_startPos distance _targetPos > 180} do {
	_startPos = [_startPos,_boundDist,_direction] call fn_trans_pos;
	_tempPos = [_startPos,_grpDist,_direction+90] call fn_trans_pos;
	
	// Overwatch
	_wp = _group2 addWaypoint [_tempPos,0];
	_wp setWaypointCompletionRadius 1;
	_wp setWaypointType "MOVE";
	_wp setWaypointBehaviour "AWARE";
	_wp setWaypointStatements ["true", "(group this) call fn_startOverwatching; ((group this) getVariable 'bounding_buddy') setVariable ['att_okayToBound',true];"];
	
	// Bounding
	_wp = _group2 addWaypoint [_tempPos,0];
	_wp setWaypointStatements ["(group this) getVariable ['att_okayToBound',false]","(group this) setVariable ['att_okayToBound',false]; (group this) call fn_startBounding; "];
	_wp setWaypointCompletionRadius 1;
	_wp setWaypointType "MOVE";
	//_wp setWaypointTimeout [1,3,4];
	
	// Overwatch
	_wp = _group1 addWaypoint [_startPos,0];
	_wp setWaypointBehaviour "AWARE";
	_wp setWaypointStatements ["true", "(group this) call fn_startOverwatching; ((group this) getVariable 'bounding_buddy') setVariable ['att_okayToBound',true];"];
	_wp setWaypointCompletionRadius 1;
	_wp setWaypointType "MOVE";
	
	// Bounding
	_wp = _group1 addWaypoint [_startPos,0];
	_wp setWaypointCompletionRadius 1;
	_wp setWaypointType "MOVE";
	_wp setWaypointStatements ["(group this) getVariable ['att_okayToBound',false]"," (group this) setVariable ['att_okayToBound',false]; (group this) call fn_startBounding;"];
	//_wp setWaypointTimeout [1,3,4];
	sleep 0.5; // Pace out the creation of the waypoints.
};

//Commit groups to final.

{
	((waypoints _group1) select (count (waypoints _group1) - 1)) setWaypointStatements ["true","(group this) call fn_startAssault"];
	((waypoints _group2) select (count (waypoints _group2) - 1)) setWaypointStatements ["true","(group this) call fn_startAssault"];

	_wp = _x addWaypoint [_targetPos,50];
	_wp setWaypointCompletionRadius 1;
	_wp setWaypointType "SAD";
	_wp setWaypointCombatMode "RED";
	_wp setWaypointBehaviour "COMBAT";
} forEach [_group1,_group2];

```

# Intragroup Bounding Scripted Waypoint
Arma 3 offers the ability to create custom scripted waypoints via its `setWaypointScript` SQF command. This allows you to attach code.


## In action
{{< youtube IarNU65wtzI >}}

# Summary
One of the key takeways that to accomplish this in SQF is unfortunately expense there is a lot of code that needs to run to monitor the group and issue actions. This may work well for small numbers of AI but would efficiently scale to big numbers. Hopefully this is something addressed by AI in a future military simulation game.