# cactus-progress-bars
Adds new progress bars to [MantisBT](http://www.mantisbt.org/), the free web-based bugtracking system.
Requires compatible theme like [cactus light](https://github.com/cactushead/cactus-light-theme) for custom CSS*

* 2018 by Cactushead

## Version

1.0 (20180117)
Tested with Mantis 2.9

## In Action

<img src="https://github.com/cactushead/cactus-progress-bars/blob/master/screenshots/cactus_progress_bars.png" alt="cactus progress bars">

* This requires alterations to a couple of MantisBT files.
1. `mantis/core/custom_field_api.php`
2. `mantis/roadmap_page.php`

* Please note that if you update MantisBT in the future, it is possible/probable that you will have to re-install these changes.

## Why Change Bootstrap Progress Bars?

### Bootstrap
Bootstrap progress bars look great and you can easily change their color and other styling attributes.
However, you can see below that their text is centred on the active percentage not across the whole bar.  This means that they all start at different horizontal positions which looks a bit odd.
<img src="https://github.com/cactushead/cactus-progress-bars/blob/master/screenshots/bootstrap_progress_bars.png" alt="bootstrap progress bars">

Other problems are that if the active percentage section (the coloured bar) is too small for the text, it will be either clipped off - or it will be in a colour that you cannot read.
<img src="https://github.com/cactushead/cactus-progress-bars/blob/master/screenshots/bootstrap_progress_bar.png" alt="bootstrap progress bar">
<img src="https://github.com/cactushead/cactus-progress-bars/blob/master/screenshots/bootstrap_progress_bars.png" alt="bootstrap progress bar clipped">

This centrally aligned text in the active percentage section does however make sense for multiple percentages in a single bar which are not needed in the MantisBT backend.

### Cactus-Progress-Bars
My version of the progress bars will show a complementary colour of text centred on each progress bar.
Additionally, if you look carefully below you can see that the text will automatically be displayed in either a light or dark colour depending on what the background colour is.
And if your text spans both the coloured percentage section as well as the lighter background - it will change the text colour for each colour (i.e. when you are close to 50%).

<img src="https://github.com/cactushead/cactus-progress-bars/blob/master/screenshots/cactus_progress_bars.png" alt="cactus progress bars">

## Setup

1. Go to the **Manage** section of your Mantis admin area
2. Select the **Install** tab
3. Type in the name **Percentage** (note the capital P) and press the **New Custom Field** button
4. Now edit the field details so that it has the following entries:-
<img src="https://github.com/cactushead/cactus-progress-bars/blob/master/screenshots/edit_custom_fields.png" alt="edit custom fields">
5. Just underneath this section - make sure to link the custom field to your desired project(s)
6. Go to the **Manage** section, then select **Manage Configuration** tab and then the **Manage Columns** Sub-tab
7. In your **All Available Columns** section you should now see the `, custom_Percentage` at the end of the list.
8. Copy this (including the comma) and paste at the end of the **View Issues Columns** list
9. Click on the **Update Columns as Global Default for All Projects** button

You should now see the sortable Percentage column with progress bars on the **View Issues** page.
Additionally, if you click on the **Roadmap** page, the progress bars there are also changed to these new ones (you may or may not see them depending on your particular setup and the progress of your project).

### Sorting
A work around for the custom percentage not being properly sorted was required as by default you would get `10, 100, 20, 30, 40, 50, 60, 70, 80, 90` so just create the custom field with `10, 20, 30, 40, 50, 60, 70, 80, 90, 99` and use the 99% value to denote 100% completion - don't worry, it *will* display as 100%


## Install

* Copy the `custom_field_api.php` file to your `mantis/core/` folder.
* Copy the `roadmap_page.php` file to your root `mantis/` folder.
* Make sure to do the **Setup** detailed above.
* Use a compatible theme like [cactus-light-theme](https://github.com/cactushead/cactus-light-theme/archive/master.zip) or manually add CSS (see below)

---

## Install Manually

### custom_field_api.php
Open the file `mantis/core/custom_field_api.php` and scroll down to about line **1505**.
look for the following line of code:-
```php echo string_display_line_links( string_custom_field_value( $p_def, $p_field_id, $p_bug_id ) );
```
Replace this line of code with the following code:-
```php // cactushead percentage progress bars
$name = $p_def['name'];
$test = custom_field_get_value( $p_field_id, $p_bug_id );

// change color of bar depending on percentage
switch(true) {
  case ($test > 70): $progrezzcolor = "progrezzcolor4"; break;
  case ($test > 50): $progrezzcolor = "progrezzcolor3"; break;
  case ($test > 30): $progrezzcolor = "progrezzcolor2"; break;
  case ($test <= 30 && $test > 0): $progrezzcolor = "progrezzcolor1"; break;
  default: $progrezzcolor = "progrezzcolor0"; break;
}

// work around for custom percentage not being properly sorted
// was: 10, 100, 20, 30, 40, 50, 60, 70, 80, 90
// now: 10, 20, 30, 40, 50, 60, 70, 80, 90, 99
// set custom field to 99 so sort works correctly - and then just display it here as 100%
if($test == 99) { $test = 100; }

// display progress bar or other custom field text
if ($name == "Percentage") {
  echo '<div class="ddiv progrezzcontainer">';
    echo '<div class="ddiv progrezzbg"></div>';
    echo '<div class="ddiv progrezz" style="width:'.$test.'%;"></div>';
    echo '<div class="ddiv progrezzcolor '.$progrezzcolor.'"></div>';
    echo '<span class="progrezztext">'.$test.'%</span>';
  echo '</div>';
} else {
  echo string_display_line_links( string_custom_field_value( $p_def, $p_field_id, $p_bug_id ) );
}
```
Save and upload to replace the existing `mantis/core/custom_field_api.php` file


### roadmap_page.php
Secondly, open the file `mantis/roadmap_page.php` and scroll down to about line **360**.
look for the following lines of code:-
```php echo '<div class="space-4"></div>';
			echo '<div class="col-md-7 col-xs-12 no-padding">';
			echo '<div class="progress progress-striped" data-percent="' . $t_progress . '%" >';
			echo '<div style="width:' . $t_progress . '%;" class="progress-bar progress-bar-success"></div>';
			echo '</div></div>';
			echo '<div class="clearfix"></div>';
```

Replace these lines of code with the following code:-
```php // cactushead percentage progress bars
// change color of bar depending on percentage
switch(true) {
  case ($t_progress > 70): $progrezzcolor = "progrezzcolor4"; break;
  case ($t_progress > 50): $progrezzcolor = "progrezzcolor3"; break;
  case ($t_progress > 30): $progrezzcolor = "progrezzcolor2"; break;
  case ($t_progress <= 30 && $t_progress > 0): $progrezzcolor = "progrezzcolor1"; break;
  default: $progrezzcolor = "progrezzcolor0"; break;
}

// cactushead - display new style percentage bar
echo '<div class="space-4"></div>';
echo '<div class="col-md-7 col-xs-12 no-padding" style="margin-bottom: 20px;">';			
  echo '<div class="ddiv progrezzcontainer">';
    echo '<div class="ddiv progrezzbg"></div>';
    echo '<div class="ddiv progrezz" style="width:'.$t_progress.'%;"></div>';
    echo '<div class="ddiv progrezzcolor progrezzcolor '.$progrezzcolor.'"></div>';
    echo '<span class="progrezztext">'.$t_progress.'%</span>';
  echo '</div>';
echo '</div>';
echo '<div class="clearfix"></div>';
```
Save and upload to replace the existing `mantis/roadmap_page.php` file


## Compatible Themes

### Cactus-Light-Theme
- Creates a flat clean and vibrant theme
- Alters side menu
- Adds theme info to footer
- No additional javascripts required

<img src="https://github.com/cactushead/cactus-light-theme/raw/master/cactus%20light.png" height="200" alt="cactus light theme for MantisBT">

[Download](https://github.com/cactushead/cactus-light-theme/archive/master.zip)

* If you want to use the progress bars in another theme then you can copy the following CSS and include it at the bottom of your `default.css` file:-
```CSS /* Progress Bar Styles
   will only apply if changes made to MantisBT files */
.ddiv {
	position: absolute;
	height: 18px;
	border-radius: 3px;
}

.progrezzcontainer {
	width: 100%;
	margin-top: 5px;
	position: relative;
}

.progrezzbg {
	background: #ddd;
	z-index: 1;
	width: 100%;
	border-radius: 3px;
}

.progrezz {
	background: black;
	z-index: 2;
	border-radius: 3px;
}

.progrezztext {
	position: absolute;
	font-family: Arial, Helvetica;
	font-size: 12px;
	mix-blend-mode: difference;
	color: white;
	z-index: 3;
	width: 100%;
	padding-top: 1px;
	text-align: center;
}

.progrezzcolor {
	mix-blend-mode: screen;
	width: 100%;
	z-index: 4;
}

.progrezzcolor0 { background-color: #999999; } /*grey progress bar*/
.progrezzcolor1 { background-color: #59A84B; } /*green progress bar*/
.progrezzcolor2 { background-color: #2A91D8; } /*blue progress bar*/
.progrezzcolor3 { background-color: #F2BB46; } /*orange progress bar*/
.progrezzcolor4 { background-color: #CA5952; } /*red progress bar*/
```

### &lt;Customising&gt;

#### Customise Colours

Simply alter the hex values of the 5 lines named `progrezzcolor0` to `progrezzcolor4`at the bottom of the CSS.

#### Customise Ranges

In the new `mantis/roadmap_page.php` file, alter the values in the case statement:-
```php case ($t_progress > 70): $progrezzcolor = "progrezzcolor4"; break;
case ($t_progress > 50): $progrezzcolor = "progrezzcolor3"; break;
case ($t_progress > 30): $progrezzcolor = "progrezzcolor2"; break;
case ($t_progress <= 30 && $t_progress > 0): $progrezzcolor = "progrezzcolor1"; break;
```
Just change the **70**, **50** and **30** values to alter the colour ranges.
i.e.
Change values to **75**, **50** and **25** to split the bar into quarters.

## ToDo

 - Create more themes


## License

Licensed under [VVL 1.33b7](http://tim-pietrusky.de/license) which means this **work** is free for all, so use it as you like.
