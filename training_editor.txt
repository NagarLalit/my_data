$(document).ready(function() {
    if($("#operationType").val() == "editTraining" ){
        $("input[name=training_type]").attr("disabled", true);
        $("#dt_a_dropdown").addClass("disabled-color");
        $("input[name=training_type]").addClass("disabled-color");
    }
});
function loadTrainingSetup() {
    window.location = BASE_URL + "training_editor_home.php?training_setup=0";

}

function loadTestSetup() {
    window.location = BASE_URL + "training_editor_home.php?training_setup=0&test_setup=1";
}

function openFilebrowser(elemId) {
    var elem = document.getElementById(elemId);
    if (elem && document.createEvent) {
        var evt = document.createEvent("MouseEvents");
        evt.initEvent("click", true, false);
        elem.dispatchEvent(evt);
        document.getElementById("upload_video").value = null;
        resetValuesFileControl();
    }
    return false;
}

function resetValuesFileControl() {
    document.getElementById("show_video_duration").innerHTML = "&nbsp;&nbsp;";
    document.getElementById("video_duration").value = "";
    document.getElementById("video_file_name").innerHTML = "Accepted file formats: mp4, avi, mkv. Max. file size: 200Mb";
}

function setValueToNull(element){
    element.value = null;
}

function fileUpload(element) {
    window.URL = window.URL || window.webkitURL;
    var video_file = element.files[0];
    if (video_file !== undefined) {
        var video_file_size = video_file.size / 1024 / 1024; // divided by 1024 to convert bytes to mega bytes.
        if (video_file_size <= 200) {
            var video = document.createElement('video');
            video.preload = 'metadata';
            video.onloadedmetadata = function() {
                window.URL.revokeObjectURL(this.src);
                var duration = video.duration;
                var date = new Date(null);
                date.setSeconds(duration);
                duration = date.toISOString().substr(11, 8);
                document.getElementById("show_video_duration").innerHTML = duration;
                document.getElementById("video_duration").value = duration;
            };
            video.src = URL.createObjectURL(video_file);
            document.getElementById("video_file_name").innerHTML = video_file.name;
            console.log(video);
            console.log(video.duration);
        } else {
            var popupsObj = new Popups();
            var msg = "You cannot upload video having size more than 200 MB";
            popupsObj.createWarningPopupSingleButton("Warning !", msg, "Close");
            document.getElementById("upload_video").value = "";
        }
    }
}

function update_values(element, elementToUpdate) {
    switch (elementToUpdate) {
        case "TRAIN_GOALS":
            document.getElementById("training_details_goals").innerHTML = element.value;
            break;
        case "TRAIN_EFFECTIVE_DATE":
            var dateInString = getFormattedDate(element.value);
            document.getElementById("training_details_effective_date").innerHTML = dateInString;
            break;
    }
}

function getFormattedDate(date) {
    date = new Date(date);
    var year = date.getFullYear();

    var month = (1 + date.getMonth()).toString();
    month = month.length > 1 ? month : '0' + month;

    var day = date.getDate().toString();
    day = day.length > 1 ? day : '0' + day;

    return day + '-' + month + '-' + year;
}

function validateTrainingSetupData() {
    var errorMessage = "";
    var trainingTitle = document.getElementById("training_title").value.trim();
    var trainingVideo = document.getElementById("upload_video").value.trim();
    var trainingDescription = document.getElementById("training_description").value.trim();
    var trainingGoal = document.getElementById("training_goal").value.trim();
    var trainingCategory = document.getElementById("training_category");
    var trainingValidFrom = document.getElementById("valid_from").value.trim();
    var trainingValidUntil = document.getElementById("valid_until").value.trim();
    var trainingVersion = document.getElementById("training_version").value.trim();
    var trainingType = document.querySelector('input[name=training_type]:checked');
    var departments = document.getElementById("department_ids").value.trim();

    if (isEmpty(trainingTitle)) {
        errorMessage += "Training title cannot be empty.<br/>";
    }
    if(document.getElementById("training_id") == null){
        if (isEmpty(trainingVideo)) {
            errorMessage += "Please upload training video.<br/>";
        }
    }
    if (isEmpty(trainingDescription)) {
        errorMessage += "Training description cannot be empty.<br/>";
    }
    if (isEmpty(trainingGoal)) {
        errorMessage += "Training goal cannot be empty.<br/>";
    }
    // if(trainingCategory.selectedIndex == 0){
    //     errorMessage += "Select training section.<br/>";
    // }
    if (isEmpty(trainingValidFrom)) {
        errorMessage += "Valid from cannot be empty.<br/>";
    }
    if (isEmpty(trainingValidUntil)) {
        errorMessage += "Valid until cannot be empty.<br/>";
    }
    var d1 = Date.parse(trainingValidUntil);
    var d2 = Date.parse(trainingValidFrom);
    if (d1 < d2) {
        errorMessage += "Valid until date cannot be less than valid from date.<br/>";
    }
    if (isEmpty(trainingVersion)) {
        errorMessage += "Training version cannot be empty.<br/>";
    }
    if (trainingType == null) {
        errorMessage += "Please select training type.<br/>";
    }
    else{
        if(trainingType.value == 2){
            if(departments == null || departments == ""){
                errorMessage += "Please select department (assign to).<br/>";
            }
        }
    }

    if (errorMessage.length > 0) {
        var popupsObj = new Popups();
        popupsObj.createGeneralPopupSingleButton(errorMessage, "OK");
        return false;
    }
    return true;
}

function isEmpty(str) {
    return (!str || 0 === str.length);
}

function disableChildDepartment(id, subDepartmentString) {
    var subDepartmentArray = subDepartmentString.split(' ');
    if ($("#dept_id_" + id).is(':checked')) {
        for (var i = 0; i < subDepartmentArray.length; i++) {
            if (i != 0) {
                $('#dept_id_' + subDepartmentArray[i]).attr("disabled", true);
            }
            $('#dept_id_' + subDepartmentArray[i]).attr('checked', true);
        }
        var selectedDepartment = $("#department_ids").val();
        if (selectedDepartment == "") {
            $("#department_ids").val(subDepartmentString);
        } else {

            var selectedDepartmentArray = selectedDepartment.split(' ');
            for (var i = 0; i < subDepartmentArray.length; i++) {
                for (var j = 0; j < selectedDepartmentArray.length; j++) {
                    if (subDepartmentArray[i] == selectedDepartmentArray[j]) {
                        selectedDepartmentArray.splice($.inArray(subDepartmentArray[i], selectedDepartmentArray), 1);
                        break;
                    }
                }
            }
            selectedDepartment = selectedDepartmentArray.join(" ");
            $("#department_ids").val(selectedDepartment + " " + subDepartmentString);
        }
    } else {
        for (var i = 1; i < subDepartmentArray.length; i++) {
            $('#dept_id_' + subDepartmentArray[i]).removeAttr("disabled");
            //$('#dept_id_' + subDepartmentArray[i]).removeAttr('checked');
            $('#dept_id_' + subDepartmentArray[i]).attr('checked', false);
        }
        var selectedDepartment = $("#department_ids").val();
        var selectedDepartmentArray = selectedDepartment.split(' ');
        for (var i = 0; i < subDepartmentArray.length; i++) {
            for (var j = 0; j < selectedDepartmentArray.length; j++) {
                if (subDepartmentArray[i] == selectedDepartmentArray[j]) {
                    selectedDepartmentArray.splice($.inArray(subDepartmentArray[i], selectedDepartmentArray), 1);
                    break;
                }
            }
        }
        selectedDepartment = selectedDepartmentArray.join(" ");
        $("#department_ids").val($.trim(selectedDepartment));
    }
}

$(document).ready(function() {
    $("#training_setup_form").on("submit", function(e) {
        $("#DATA_LOADER").show();
        $("#DATA_LOADER").html("");
        var popupsObj = new Popups();
        popupsObj.createGeneralPopupSingleButton('Uploading video...', "OK"); 
        $("#p_custom_message_body").append('<div id="progressBar">0%</div>');
        var positiveBtn = popupsObj.getPositiveButtonRef();
        $(positiveBtn.parentNode).hide();
        var isValid = validateTrainingSetupData();
        e.preventDefault();
        //tinyMCE.triggerSave();
      //  $('#training_description').val( tinymce.get('training_description').getContent() );
       // console.log($('#training_description').val());
        if (isValid) {
            $.ajax({
                type: "POST",
                url: "/bb-training-system/controllers/training_controller.php",
                data: new FormData(document.getElementById('training_setup_form')),

                // Tell jQuery not to process data or worry about content-type
                // You *must* include these options!
                cache: false,
                contentType: false,
                processData: false,
                success: function(response) {
                    console.log(response);
                    try{
                        response = JSON.parse(response);
                        if (response.STATUS == "200") {
                            //alert(response.MESSAGE);
                            $("#p_custom_message_body").html(response.MESSAGE);
                            $("#DATA_LOADER").hide();
                            $(positiveBtn.parentNode).show();
                            $(positiveBtn).on('click', function() {
                                // POSITIVE BUTTON ACTION
                                window.location = BASE_URL + "training_editor_home.php?training_setup=" + response.TRAINING_ID + "&test_setup=1";
                            });
                            //window.location = BASE_URL + "training_editor_home.php?training_setup=" + response.TRAINING_ID + "&test_setup=1";
                        }
                        else if(response.STATUS == "201"){
                            if(response.RESULT.length > 0){
                                var errorMessage = "";
                                for(var errors = 0; errors < response.RESULT.length; errors++){
                                    errorMessage += response.RESULT[errors].MESSAGE + "<br/>";
                                }
                                var popupsObj = new Popups();
                                popupsObj.createGeneralPopupSingleButton(errorMessage, "OK");
                                $("#DATA_LOADER").hide();
                            }
                        }
                        else if(response.STATUS == "300"){
                            window.location = BASE_URL;
                        } else {
                            console.log(response);
                        }
                    }
                    catch(e){
                        console.log(e);
                        console.log("Response is not in json format");
                    }
                },
                xhr: function(){
                    // get the native XmlHttpRequest object
                    var xhr = $.ajaxSettings.xhr() ;
                    // set the onprogress event handler
                    xhr.upload.onprogress = function(evt){ 
                        $("#DATA_LOADER").html("");
                        console.log('progress', evt.loaded/evt.total*100); 
                        var percentageCompleted = evt.loaded/evt.total*100;
                        updateUploadPercentage(parseInt(percentageCompleted));
                        if(percentageCompleted == 100){
                            $("#p_custom_message_body").html('<p>Storing training data</p><img src="media/images/loading.gif">');
                        }
                    } ;
                    // set the onload event handler
                    xhr.upload.onload = function(){ console.log('DONE!') } ;
                    // return the customized object
                    return xhr ;
                }
                    // // Custom XMLHttpRequest
                    // xhr: function() {
                    //     var myXhr = $.ajaxSettings.xhr();
                    //     if (myXhr.upload) {
                    //         // For handling the progress of the upload
                    //         myXhr.upload.addEventListener('progress', function(e) {
                    //             if (e.lengthComputable) {
                    //                 $('progress').attr({
                    //                     value: e.loaded,
                    //                     max: e.total,
                    //                 });
                    //             }
                    //         } , false);
                    //     }
                    //     return myXhr;
                    // },
            });
        }
        else{
            $("#DATA_LOADER").hide();
        }
    });
});

function showTrainingDetails(trainingId, trainingVersion, Duration, effectiveDate, reviewDate) {
    $("#training_overview tr").removeClass("td-selected");
    $("#tr" + trainingId).addClass("td-selected");
    var goal = $('#goal_' + trainingId).val();
    var userName = $('#user_name').val();
    $('#training_details_author').text(userName);
    $('#training_details_version').text(trainingVersion);
    $('#training_details_duration').text(Duration);
    $('#training_details_effective_date').text(effectiveDate);
    $('#training_details_review_date').text(reviewDate);
    $('#training_details_goals').html(goal);
}
function checkTrainingType()
{
    if($('input[name=training_type]:checked').val()==1)
    {
        $("#dt_a_dropdown").addClass("disabled-color");
    }
    else
    {
        $("#dt_a_dropdown").removeClass("disabled-color");
    }    
}

function updateUploadPercentage(progress) {
    var elem = document.getElementById("progressBar");
    elem.style.width = progress + '%'; 
    elem.innerHTML = progress * 1 + '%';
}