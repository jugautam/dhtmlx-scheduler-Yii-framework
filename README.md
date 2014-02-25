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

Inside Body include this div:

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

Custom Form to include inside body tag. Lightbox popup this form

<div id="my_form">
    <?php echo CHtml::label('Title', 'text'); ?>
    <?php echo CHtml::textField('text', ''); ?>

    <br><br>
    <input type="button" name="save" value="Save" id="save" style='width:100px;' onclick="save_form()">
    <input type="button" name="close" value="Close" id="close" style='width:100px;' onclick="close_form()">
    <input type="button" name="delete" value="Delete" id="delete" style='width:100px;' onclick="delete_event()">
</div>


Inside Default Controller

/**
this action is used to render view
*/
public function actionIndex()
{
    $client_data = $this->getClients();
    $user_data = $this->getUsers();

    $this->render('index', array('client_data' => $client_data, 'user_data' => $user_data));
}

/**
* load the data to populate the calendar, called from view(index.php)
*/
public function actionScheduler_data()
{
    $dbType = "PHPYii";

    $data = Calendar::model()->findAll();

    $scheduler = new SchedulerConnector($data, "PHPYii");
    
    # first parameter - default for Yii
    # second parmeter - id is primary key
    # third parameter - start_date, end_date, text // include as many to get data in client side
    $scheduler->configure("-", "id", "start_date, end_date, text, assignedTo"); // first parameter is default for yii

    // format data before populating data in calendar
    function formatTimestamp($action) {

        $ds = $action->get_value("start_date");
        $de = $action->get_value("end_date");

        $action->set_value("start_date", date('Y-m-d H:i', $ds));
        $action->set_value("end_date", date('Y-m-d H:i', $de));
    }

    $scheduler->event->attach("beforeRender","formatTimestamp");


    $scheduler->render();
}

/**
* save the data to database
*/
public function actionScheduler_save()
{
    //this action will be used for data saving
    //we need to provide a model as first parameter

    $data = Calendar::model();

    $scheduler = new SchedulerConnector($data, "PHPYii");
    $scheduler->configure("-", "id", "start_date, end_date, text, assignedTo");

    function beforeSaving($action) {

        $userId = Yii::app()->user->id;

        $ds = $action->get_value("start_date");
        $de = $action->get_value("end_date");

        $action->set_value("start_date", strtotime($ds));
        $action->set_value("end_date", strtotime($de));
        
        # Do custom saving of data
        
        $action->success(); // after saving terminate the process
        }

    }

    $scheduler->event->attach("beforeProcessing","beforeSaving"); // event triggered before Processing

    $scheduler->render();
}
