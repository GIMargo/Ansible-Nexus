Создаём необходимые директории:
mkdir -p ~/infra/roles/nexus/{handlers,tasks,templates,defaults}

Содержимое директорий (кроме директории templates) можно взять из данного примера

В директории: 
/infra/roles/nexus/defaults
хранятся локальные переменные

На что нужно обратить внимание:
1. В файле .../tasks/main_install_upgrade.yml в таске "Replace JAVA_HOME_OVERRIDE" переменной INSTALL4J_JAVA_HOME_OVERRIDE присваивается то же самое значение, 
что лежит в файле /opt/nexus/bin/nexus
У меня это:
INSTALL4J_JAVA_HOME_OVERRIDE=/usr/lib/jvm/java-8-openjdk-amd64
2. В том же файле в таске "Add user nexus" у пользователя nexus должна быть та же самая оболочка, что и в файле /etc/passwd
У меня это:
nexus:x:1001:1001::/opt/nexus:/usr/sbin/nologin
т.е.: /usr/sbin/nologin
3. В файле .../tasks/install_preq.yml в таске "Add non-free system repository" в переменную repo кладём то же, что лежит в файле /etc/apt/sources.list
У меня это:
deb http://deb.debian.org/debian/ bookworm main non-free-firmware non-free contrib

В infra/hosts.txt добавляем nexus (IP тот же)
Создаём новый плэйбук на основе прошлого (nginx):
cp nginx.yml nexus.yml

В нём (~/infra/nexus.yml) добавляем после -nginx: -nexus

Копируем шаблон:
cp /etc/systemd/system/nexus.service ~/infra/roles/nexus/templates/nexus.service.j2
Меняем в полученном файле User=nexus на User={{ nexus_user }}

Копируем шаблон:
cp /etc/nginx/sites-available/nexus ~/infra/roles/nexus/templates/nexus.j2
Меняем там в строке, где server_name, nexus.develop.local на {{ nexus_domain }}

Запуск playbook-a:
ansible-playbook -i hosts.txt nexus.yml -C -K -D
(выполняется в директории infra, так как именно в ней лежат эти файлы)
Флаг -С сигнализирует о том, что плэйбук запускается с проверкой, никакие файлы в таком режиме скачиваться и создаваться не будут, как и изменения вноситься

Если всё отработало правильно, вам должны предложить не более 8 изменений (каждое нужно проверить, что ничего не поломается)
Допустимые изменения: добавился/убрался пробел, знак табуляции, поменялся IP-адрес, поменялось значение переменных по типу Xms
Когда убедились, что ничего не сломаем, можно выполнить без чека:
ansible-playbook -i hosts.txt nexus.yml -K -D

Можно передать переменную и её значение с помощью флага -e (extra)
Если где-то уже была переменная с таким именем, то её значение изменится на переданное
ansible-playbook -i hosts.txt nexus.yml -C -K -D -e "nexus_upgrade=true"

С помощью данной переменной можно проверить, как сработает установка новой версии nexus
Убедились, что всё работает - выполняем без чека:
ansible-playbook -i hosts.txt nexus.yml -K -D -e "nexus_upgrade=true"

На следующей паре будем проверять "чистую" (с нуля) установку nexus
Шаги:
systemctl stop nexus
rm -rf /opt/nexus*
cp -R /opt/sonatype-work /
rm -rf /opt/sonatype-work/
ansible-playbook -i hosts.txt nexus.yml -C -K -D

