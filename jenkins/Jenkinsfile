                   // GLOBAL_CONSTANTS

def C_APP_NAME = 'exntu-test-showcase'; // openshift project 내의 app 이름
def C_GIT_SERVER = 'https://github.com/superpeace/spring-mvc-showcase.git'; // git 서버 주소
def C_MAIL_FROM = 'sekim@bliex.com'; // 메일 보내는 사람설정
def C_MAIL_TO = 'sekim@bliex.com,sekim@renosoft.kr'; // 메일 받는 사람(다중값 지원, 콤마 구분
def C_MAVEN_NAME = 'M3'; // jenkins 내에 메이븐 툴 이름
def C_PRD_JENKINS = 'https://jenkins-pipeline-prd.b9ad.pro-us-east-1.openshiftapps.com';   // productio jenkins url
def C_PRD_JENKINS_HOOK_TOKEN = '1234567890';

// GLOBAL_FUNCTION
def debug_global = {
    // 파이프라인의 글로벌 변수값을 출력해준다.
    echo "-----------------------------"
    echo "       global constants"
    echo "-----------------------------"
    echo "C_APP_NAME                   ${C_APP_NAME}"
    echo "C_GIT_SERVER                 ${C_GIT_SERVER}"
    echo "C_MAIL_FROM                  ${C_MAIL_FROM}"
    echo "C_MAIL_TO                    ${C_MAIL_TO}"
    echo "C_MAVEN_NAME                 ${C_MAVEN_NAME}"

    echo "-----------------------------"
    echo "       global parameter"
    echo "-----------------------------"
    params.each {
        pname, pvalue ->  echo "${pname}       :       ${pvalue}"
    };
    echo "-----------------------------"
    echo "       global env"
    echo "-----------------------------"
    echo "BRANCH_NAME                   ${env.BRANCH_NAME}"
    echo "CHANGE_ID                     ${env.CHANGE_ID}"
    echo "CHANGE_URL                    ${env.CHANGE_URL}"
    echo "CHANGE_TITLE                  ${env.CHANGE_TITLE}"
    echo "CHANGE_AUTHOR                 ${env.CHANGE_AUTHOR}"
    echo "CHANGE_AUTHOR_DISPLAY_NAME    ${env.CHANGE_AUTHOR_DISPLAY_NAME}"
    echo "CHANGE_AUTHOR_EMAIL           ${env.CHANGE_AUTHOR_EMAIL}"
    echo "CHANGE_TARGET                 ${env.CHANGE_TARGET}"
    echo "BUILD_NUMBER                  ${env.BUILD_NUMBER}"
    echo "BUILD_ID                      ${env.BUILD_ID}"
    echo "BUILD_DISPLAY_NAME            ${env.BUILD_DISPLAY_NAME}"
    echo "JOB_NAME                      ${env.JOB_NAME}"
    echo "JOB_BASE_NAME                 ${env.JOB_BASE_NAME}"
    echo "BUILD_TAG                     ${env.BUILD_TAG}"
    echo "EXECUTOR_NUMBER               ${env.EXECUTOR_NUMBER}"
    echo "NODE_NAME                     ${env.NODE_NAME}"
    echo "NODE_LABELS                   ${env.NODE_LABELS}"
    echo "WORKSPACE                     ${env.WORKSPACE}"
    echo "JENKINS_HOME                  ${env.JENKINS_HOME}"
    echo "JENKINS_URL                   ${env.JENKINS_URL}"
    echo "BUILD_URL                     ${env.BUILD_URL}"
    echo "JOB_URL                       ${env.JOB_URL}"

    echo "-----------------------------"
    echo "       global currentBuild"
    echo "-----------------------------"
    echo "currentBuild.number             ${currentBuild.number}"
    echo "currentBuild.result             ${currentBuild.result}"
    echo "currentBuild.currentResult      ${currentBuild.currentResult}"
    echo "currentBuild.displayName        ${currentBuild.displayName}"
    echo "currentBuild.description        ${currentBuild.description}"
    echo "currentBuild.id                 ${currentBuild.id}"
    echo "currentBuild.timeInMillis       ${currentBuild.timeInMillis}"
    echo "currentBuild.startTimeInMillis  ${currentBuild.startTimeInMillis}"
    echo "currentBuild.duration           ${currentBuild.duration}"
    echo "currentBuild.durationString     ${currentBuild.durationString}"
    echo "currentBuild.previousBuild      ${currentBuild.previousBuild}"
    echo "currentBuild.nextBuild          ${currentBuild.nextBuild}"
    echo "currentBuild.absoluteUrl        ${currentBuild.absoluteUrl}"
    echo "currentBuild.buildVariables     ${currentBuild.buildVariables}"
    echo "currentBuild.changeSets         ${currentBuild.changeSets}"
    echo "currentBuild.rawBuild           ${currentBuild.rawBuild}"
}

def getContainedLogLine = {
    pContainedWord ->
    def alljob = Jenkins.getInstance().getItemByFullName(env.JOB_NAME).getAllJobs()
    def interAlljob = alljob.iterator()
    for (job in interAlljob) {
        if (env.JOB_NAME == job.getDisplayName()) {
            def log_reader = job.getLastBuild().getLogReader();
            //echo "DEBUG "
            //echo job.getLastBuild().getLogFile().getAbsolutePath()
            while (true) {
                String line = log_reader.readLine();
                if (line != null && line.contains(pContainedWord)) {
                    def(a, b) = line.split(pContainedWord)
                    return b;
                    break;
                }
            }
            log_reader.close()
            break;
        }
    }
    return ""
}

def getAllLogLine = {

    def alljob = Jenkins.getInstance().getItemByFullName(env.JOB_NAME).getAllJobs()
    def interAlljob = alljob.iterator()
    def allLog = new StringBuilder();

    for (job in interAlljob) {
        if (env.JOB_NAME == job.getDisplayName()) {
            def log_reader = job.getLastBuild().getLogReader();
            //echo "getAllLogLine DEBUG "
            //echo job.getLastBuild().getLogFile().getAbsolutePath()
            String line = log_reader.readLine()
            while (line != null) {
                allLog.append(line);
                line = log_reader.readLine();
            }
            log_reader.close()
            break;
        }
    }
    //return allLog.toString().replaceAll('[Pipeline]','\n\n[Pipeline]');
    return allLog.toString();
}

try {
    timeout(time: 20, unit: 'MINUTES') {
        openshift.withCluster() {

                echo "START ${openshift.project()} Project CI / CD pipeline!"
                echo "Timeout 20 minute"

                stage('Test stage') {
                    echo "-----------------------------"
                    echo "       Stage - Test Server test"
                    echo "-----------------------------"
                    debug_global()
                    echo "소스 체크아웃  git source from ${C_GIT_SERVER}"
                    git C_GIT_SERVER
                    // goal : git revision tag 가지고 오기
                    def git_last_revision = getContainedLogLine("git checkout -f ");
                    echo "현재 Git revision : ${git_last_revision}"

                    // goal : revision tag 값의 openshift image가 있는지 체크 한다. 없으면 패스
                    def check_image_exist = openshift.selector("is", git_last_revision)
                    if (check_image_exist.exists()) {
                        // 이미지가 존재하면 test 하지 않고 종료한다.


                        C_MAIL_TO.split(",").each{

                            mail (

                                from : C_MAIL_FROM,
                                to: it,
                                subject: "실패 : 파이프라인 ${currentBuild.rawBuild}",
                                body : email_body,

                            );

                        }


                    } else {
                        // 이미지가 존재하지 않으면 즉 git에 새로운 추가가 있다면  소스를 test한다.
                        withMaven(
                            maven: C_MAVEN_NAME,
                        ) {
                            sh "mvn test"
                        }

                        def new_test_result = getContainedLogLine("INFO] BUILD ");
                        echo "new_test_result : ${new_test_result}"
                        if (new_test_result != "SUCCESS") {
                            // 테스트가 실패하면 실패 노티를 보낸다.
                            def email_body = "현재 git revision ${git_last_revision} 소스 junit test가 실패하였습니다. "
                            send_mail("실패 : 파이프라인 ${currentBuild.rawBuild}", email_body)
                        } else {
                            // 테스트가 성공하면 새로운 이미지를 만들고
                            stage('Build image') {
                                echo "-----------------------------"
                                echo "       Stage - Deploy test"
                                echo "-----------------------------"
                                echo "Merge - git + container "
                                def new_build_object = openshift.startBuild(C_APP_NAME);
                                echo "Describe - buildConfig "
                                new_build_object.describe();
                                //new_build_object.untilEach(1) {
                                //    return (it.object().status.phase == "Complete")
                                //}

                                //echo "tagging image with git revision"  // << tag은 이미지 생성 시간 때문에 싱크가 맞지 않음
                                //openshift.tag("${openshift.project()}/${C_APP_NAME}:lastest", "${openshift.project()}/${C_APP_NAME}:${git_last_revision}")
                                //echo "labeling image with git revision"
                                //openshift.selector("is","${C_APP_NAME}").label( [ git_revision :'${git_last_revision}'], "--overwrite" )
                                def email_body = "현재 git revision ${git_last_revision} 소스 이미지를 생성했고 test 서버에 배치하였습니다. \n "
                                email_body    += "현재 이미지를 production 으로 빌드하려면 아래 url을 클릭하세요! \n"
                                email_body    += "${C_PRD_JENKINS}/job/pipeline-prd/build?token=${C_PRD_JENKINS_HOOK_TOKEN}"


                                C_MAIL_TO.split(",").each{


                                    mail (

                                        from : C_MAIL_FROM,
                                        to: it,
                                        subject: "성공 : 파이프라인 ${currentBuild.rawBuild}",
                                        body : email_body,

                                    );

                                }

                            }
                        }
                    }
                    //debug_global()
                }

        }
    }

} catch (err) {
    currentBuild.result = 'FAILURE'

    def email_body = "에러 발생 내용 \n"
    email_body += "${err}"

    C_MAIL_TO.split(",").each{


        mail (

            from : C_MAIL_FROM,
            to: it,
            subject: "성공 : 파이프라인 ${currentBuild.rawBuild}",
            body : email_body,

        );

    }

    throw err
}