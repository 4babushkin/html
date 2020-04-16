
properties([disableConcurrentBuilds()])


pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        timestamps()
    }


	stages {  

		stage ('statyc-syte - Checkout') {
		checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'git@github.com:4babushkin/html.git']]]) 
		}

		stage ('statyc-syte - Build') {
		sshagent (credentials: ['githab']) {
				// Shell build step
			sh ''' 
			echo "------------------- env ----------------------"
			env|sort
			echo "------------------- end env ----------------------" 
			'''		// Shell build step
			sh ''' 
			#настройки по умолчанию
			HOST='jenkins.babushkin.tk'
			USER='vova'
			# путь до нашего приложения
			DOC_ROOT="/home/$USER/html"
			# имя папки на локальном компьютере с приложением
			LOCAL_APP_FOLDER=$JOB_NAME
			SSH_HOST="$USER@$HOST"

			################################
			# последний тэг   0.2
			LAST_TAG=$(git describe --abbrev=0 | sed 's/^.//')
			# хэш последнего тега   cc2e405d0d8cace925c950694694aaff209ef6a7
			LAST_TAG_COMMIT_HASH=$(git rev-list --tags --max-count=1)

			CURRENT_VERSION_HASH=$(git rev-parse HEAD)
			#################################


			## Проверка на изменения 
			echo "присваиваем переменной последний хеш"
			if [ $(ssh $SSH_HOST test -f $DOC_ROOT/last_hash && echo 1) ]; then 
			SERVER_VERSION_HASH=$(ssh $SSH_HOST cat $DOC_ROOT/last_hash)
			else
			SERVER_VERSION_HASH=0
			fi
			####

			### если изменения на сервере и в комите есть то деплоим
			if [ $CURRENT_VERSION_HASH = $SERVER_VERSION_HASH ]; then
			echo "Strings are equal, nothing to do"
			else

			echo "************************Strings are not equal, do ..................."
			cd ..
			echo '* Создаем архив...'
			tar -czf $JOB_NAME.tar.gz $JOB_NAME 


			echo '* Копируем архив на сервер... затем локально удалим ...'
			scp -rp ./$JOB_NAME.tar.gz $SSH_HOST:$DOC_ROOT/$JOB_NAME.tar.gz 
			rm -rf $JOB_NAME.tar.gz

			echo '* создадим файл с хешем последнегоо коммита для отслеживания были ли изменения... '
			echo $CURRENT_VERSION_HASH > last_hash
			scp -rp ./last_hash $SSH_HOST:$DOC_ROOT/last_hash 


			echo '* Распаковываем архив на серверe...'
			ssh $SSH_HOST "mkdir -p $DOC_ROOT/$LAST_TAG/; cd $DOC_ROOT; tar -xzf $JOB_NAME.tar.gz -C $DOC_ROOT/$LAST_TAG 2> /dev/null && rm -rf $DOC_ROOT/$JOB_NAME.tar.gz"


			echo '* Символьная ссылка'
			ssh $SSH_HOST "sudo rm -r $DOC_ROOT/www; ln -sfn $DOC_ROOT/$LAST_TAG/$JOB_NAME $DOC_ROOT/www"

			fi 
			''' 
		}
		}
	}

}