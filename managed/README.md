# Overview

This is the managed portion of the gta vision export code. This gets information from the game's scripting interface and the native plugin and uploads it to a postgres (postgis) database.

## Requirements
* GTAVisionNative (runtime)
* ScriptHookV SDK
* ScriptHookVDotNet
* VAutodrive
* NativeUI
* others managed by nuget

## Building
First go through the refereces in visual studio and update the paths for the non-nuget dependencies. These dependencies will usually live in your GTAV ddirectory. Then simply build the GTAVisionExport project and copy the resulting files into {gtav directory}/Scripts.

There is ScriptHookVDotNet in references, but it is deprecated. Use ScriptHookVDotNet2 instead of ScriptHookVDotNet.

Probably, you will need to add the System.Management dependency for it to work.

## Database config
In order the connect to the database the managed plugins needs to know your database information. 

For that, create `GTAVision.ini` file in your scripts directory with following content:
```ini
[Database]
ConnectionString=<npgsql connection string>
```

The format of the conenction can be found at http://www.npgsql.org/doc/connection-string-parameters.html

Example config for localhost:
```ini
[Database]
ConnectionString=Server=127.0.0.1;Port=5432;Database=postgres;User Id=postgres;Password=postgres;
```

## Database schema

```sql

create type detection_type AS ENUM ('background', 'person', 'car', 'bicycle');

create type detection_class AS ENUM ('Unknown',   'Compacts',   'Sedans',   'SUVs',   'Coupes',   'Muscle',   'SportsClassics',   'Sports',   'Super',   'Motorcycles',   'OffRoad',   'Industrial',   'Utility',   'Vans',   'Cycles',   'Boats',   'Helicopters',   'Planes',   'Service',   'Emergency',   'Military',   'Commercial',   'Train');

create type weather AS ENUM ('Unknown', 'ExtraSunny', 'Clear', 'Clouds', 'Smog', 'Foggy', 'Overcast', 'Raining', 'ThunderStorm', 'Clearing', 'Neutral', 'Snowing', 'Blizzard', 'Snowlight', 'Christmas', 'Halloween');

create table detections
(
	detection_id serial not null
		constraint detections_pkey
			primary key,
	snapshot_id integer,
	type detection_type,
	pos geometry(PointZ),
	bbox box,
	class detection_class default 'Unknown'::detection_class,
	handle integer default '-1'::integer,
	best_bbox box,
	best_bbox_old box,
	bbox3d box3d,
	rot geometry,
	coverage real default 0.0
)
;


create table runs
(
	run_id serial not null
		constraint runs_pkey
			primary key,
	runguid uuid,
	archivepath text,
	localpath text,
	session_id integer default 1,
	instance_id integer default 0
)
;


create table sessions
(
	session_id serial not null
		constraint sessions_pkey
			primary key,
	name text
		constraint sessions_name_key
			unique,
	start timestamp with time zone,
	"end" timestamp with time zone
)
;

alter table runs
	add constraint runs_session_fkey
		foreign key (session_id) references sessions
			on delete cascade
;

create table snapshots
(
	snapshot_id serial not null
		constraint snapshots_pkey
			primary key,
	run_id integer
		constraint snapshots_run_fkey
			references runs
				on delete cascade,
	version integer,
	imagepath text,
	timestamp timestamp with time zone,
	timeofday time,
	currentweather weather,
	camera_pos geometry(PointZ),
	camera_direction geometry,
	camera_fov real,
	view_matrix double precision[],
	proj_matrix double precision[],
	processed boolean default false not null,
	width integer,
	height integer
)
;


alter table detections
	add constraint detections_snapshot_fkey
		foreign key (snapshot_id) references snapshots
			on delete cascade
;

create table instances
(
	instance_id serial not null
		constraint isntances_pkey
			primary key,
	hostname text,
	instanceid text
		constraint instanceid_uniq
			unique,
	instancetype text,
	publichostname text,
	amiid text,
	constraint instance_info_uniq
		unique (hostname, instanceid, instancetype, publichostname, amiid)
)
;

alter table runs
	add constraint runs_instance_fkey
		foreign key (instance_id) references instances
;

create table snapshot_weathers
(
	weather_id serial not null
		constraint snapshot_weathers_pkey
			primary key,
	snapshot_id integer
		constraint snapshot_weathers_snapshot_id_fkey
			references snapshots
				on delete cascade,
	weather_type weather,
	snapshot_page integer
)
;

create table uploads
(
	id serial not null
		constraint uploads_pkey
			primary key,
	bucket text,
	key text,
	uploadid text
)
;

create table datasets
(
	dataset_id serial not null
		constraint datasets_pkey
			primary key,
	dataset_name text,
	view_name text
)
;

create table systems
(
	system_uuid uuid not null
		constraint systems_pkey
			primary key,
	vendor text,
	dnshostname text,
	username text,
	systemtype text,
	totalmem integer
)
;

create table system_graphics
(
	system_graphic_id serial not null
		constraint system_graphics_pkey
			primary key,
	deviceid text,
	adaptercompatibility text,
	adapterdactype text,
	adapterram integer,
	availability integer,
	caption text,
	description text,
	driverdate timestamp with time zone,
	driverversion text,
	pnpdeviceid text,
	name text,
	videoarch integer,
	memtype integer,
	videoprocessor text,
	bpp integer,
	hrez integer,
	vrez integer,
	num_colors integer,
	cols integer,
	rows integer,
	refresh integer,
	scanmode integer,
	videomodedesc text
)
;


```

## Copying compiled files to GTA V
After you compile the GTAVisionExport, copy compiled files from the `path to GTAVisionExport/managed/GTAVisionExport/bin/Release` to `path to GTA V/scripts`.
Content of `scripts` directory should be following: 
- AWSSDK.dll
- BitMiracle.LibTiff.NET.dll
- BitMiracle.LibTiff.NET.xml
- gdal_csharp.dll
- GTAVision.ini
- GTAVisionExport.dll
- GTAVisionUtils.dll
- GTAVisionUtils.dll.config
- INIFileParser.dll
- INIFileParser.xml
- MathNet.Numerics.dll
- MathNet.Numerics.xml
- Microsoft.Extensions.DependencyInjection.Abstractions.dll
- Microsoft.Extensions.DependencyInjection.Abstractions.xml
- Microsoft.Extensions.Logging.Abstractions.dll
- Microsoft.Extensions.Logging.Abstractions.xml
- Microsoft.Extensions.Logging.dll
- Microsoft.Extensions.Logging.xml
- NativeUI.dll
- Npgsql.dll
- Npgsql.xml
- ogr_csharp.dll
- osr_csharp.dll
- SharpDX.dll
- SharpDX.Mathematics.dll
- SharpDX.Mathematics.xml
- SharpDX.xml
- System.Runtime.InteropServices.RuntimeInformation.dll
- System.Threading.Tasks.Extensions.dll
- System.Threading.Tasks.Extensions.xml
- VAutodrive.dll
- VAutodriveConfig.xml
- VCommonFunctions.dll
- YamlDotNet.dll
- YamlDotNet.xml


## Verifying it loaded correctly

To verify all plugins loaded, see the `ScriptHookVDotNet2.log` and search for this line:
```
[23:02:26] [DEBUG] Starting 10 script(s) ...
```

If less than 10 scripts loaded, you have problem.

## Usage

In the game, turn off the HUD, MSAA and Radar.

Turn the plugin on by "Page Up" in the game.

Then, collect data by pressing "N" key.