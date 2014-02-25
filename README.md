dhtmlx-scheduler-Yii-framework
==============================

This is the module to integrate with Yii framework. To integrate with Yii you need to have dhtmlx plugin (download latest package at http://dhtmlx.com/docs/download.shtml) in project root directory. You need to copy the scheduler module inside you modules directory. 

The plugin requied files included in DefaultController, the path has been adjusted from modules directory

require_once(dirname(__FILE__) . "/../../../../dhtmlx/connector/db_phpyii.php");
require_once(dirname(__FILE__) . "/../../../../dhtmlx/connector/scheduler_connector.php");

In view default/index.php, include the script and stylesheet

<script src="<?php echo Yii::app()->baseUrl . '/dhtmlx/dhtmlxscheduler.js' ?>" type="text/javascript" charset="utf-8"></script>
    <script src="<?php echo Yii::app()->baseUrl . '/dhtmlx/ext/dhtmlxscheduler_timeline.js' ?>" type="text/javascript" charset="utf-8"></script>
    /** include this script only if you want day clickable link
    * remove the configuration if you dont want clickable link on day:
    * scheduler.config.active_link_view = "day";
    */
    <script src="<?php echo Yii::app()->baseUrl . '/dhtmlx/ext/dhtmlxscheduler_active_links.js' ?>" type="text/javascript" charset="utf-8"></script>
    <script src="<?php echo Yii::app()->baseUrl . '/dhtmlx/ext/dhtmlxscheduler_pdf.js' ?>" type="text/javascript" charset="utf-8"></script>

<link rel="stylesheet" href="<?php echo Yii::app()->baseUrl . '/dhtmlx/dhtmlxscheduler.css' ?>" type="text/css" title="no title" charset="utf-8">

This script file is included to configure the dhtmlx scheduler plugin and is called on body onload:

<script>

function init() {
    
    //==================
    //Configuration
    //=================
    var userList = <?php echo $user_data ?>;
    var clientList = <?php echo $client_data ?>;

    scheduler.config.prevent_cache = true;
    scheduler.config.multi_day = true;
    scheduler.config.active_link_view = "day"; /* this makes day link clickable */
    scheduler.config.xml_date="%Y-%m-%d %H:%i"; /* format the date */
    scheduler.config.first_hour = 7;
    scheduler.config.last_hour = 23;
    scheduler.locale.labels.timeline_tab = "Users";
    scheduler.config.details_on_create=true;
    scheduler.config.details_on_dblclick=true;

    scheduler.createTimelineView({
        name:	"timeline",
        x_unit:	"minute",
        x_date:	"%H:%i",
        x_step:	60,
        x_size: 12,
        x_start: 8,
        x_length: 24,
        y_unit:	userList, // populate the user data
        y_property:	"assignedTo", // this property is used for userview
        render:"bar"
    });

    //===============
    //Data loading
    //===============

   scheduler.init('scheduler_here',new Date(), "week"); // initialize the scheduler once

    scheduler.load("index.php?r=scheduler/default/scheduler_data"); // load data to populate the calendar
    
    /** synchronize the client and sever side data 
      * call the server side for saving the client data
    */
    var dp = new dataProcessor("index.php?r=scheduler/default/scheduler_save");
    dp.init(scheduler);
}


//===============
// custom lightbox
// evnt triggered on open and close and on save button click
//===============

var html = function(id) {return document.getElementById(id);}; //just a helper

  // on lightbox open
  scheduler.showLightbox = function(id) {
      var new_event = scheduler.getState().new_event; // if the event is new return ids otherwise return null
      var ev = scheduler.getEvent(id); // instance of the event
      
      scheduler.startLightbox(id, html("my_form")); // custom form to load

      html("text").focus(); // which element to focus on lightbox popup
      
      // do whatever before popup
      

  };

  function save_form() {

      var ev = scheduler.getEvent(scheduler.getState().lightbox_id);
      ev.text = html("text").value; // return the form value
      
      // manipulate client side data before saving
      scheduler.endLightbox(true, html("my_form")); // closes the lightbox
  }
  
  function close_form() {
      scheduler.endLightbox(false, html("my_form"));
  }

  function delete_event() {
      var event_id = scheduler.getState().lightbox_id;
      scheduler.endLightbox(false, html("my_form"));
      scheduler.deleteEvent(event_id);
  }
    
</script>

In Body 

<div id="scheduler_here" class="dhx_cal_container" style='padding:550px;'>
    <div class="dhx_cal_navline">
        <div id="export_pdf" class="dhx_cal_export pdf" onclick="scheduler.toPDF('http://dhtmlxscheduler.appspot.com/export/pdf', 'color')" title="Export to PDF"> </div>
        <div class="dhx_cal_prev_button">&nbsp;</div>
        <div class="dhx_cal_next_button">&nbsp;</div>
        <div class="dhx_cal_today_button"></div>
        <div class="dhx_cal_date"></div>
        <div class="dhx_cal_tab" name="day_tab" style="right:204px;"></div>
        <div class="dhx_cal_tab" name="week_tab" style="right:140px;"></div>
        <div class="dhx_cal_tab" name="month_tab" style="right:76px;"></div>
        <?php if (!$isTechUser) {?>
            <div class="dhx_cal_tab" name="timeline_tab" style="right:280px;"></div>
        <?php }?>

    </div>
    <div class="dhx_cal_header"></div>
    <div class="dhx_cal_data"></div>
</div>
