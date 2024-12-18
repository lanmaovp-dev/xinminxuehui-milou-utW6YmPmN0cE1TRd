
目录* [一、搭建 Neo4j 图数据库](https://github.com)
	+ [1、方式选择](https://github.com)
	+ [2、Dockerfile\+docker\-compose部署neo4j容器](https://github.com)
		- [2\.1、更新 yum 镜像源](https://github.com)
		- [2\.2、安装 docker\-ce 社区版](https://github.com)
		- [2\.3、配置镜像加速](https://github.com)
		- [2\.4、安装 Docker Compose](https://github.com)
			* [2\.4\.1、下载 Docker Compose 二进制包](https://github.com)
			* [2\.4\.2、设置可执行权限](https://github.com)
			* [2\.4\.3、查看版本](https://github.com)
		- [2\.5、创建目录结构](https://github.com)
		- [2\.6、编写neo4j.conf配置文件](https://github.com)
		- [2\.7、编写 dockerfile 文件](https://github.com)
		- [2\.8、构建ne4j容器镜像](https://github.com)
		- [2\.9、编写docker\-compose.yaml文件](https://github.com)
		- [2\.10、运行docker\-compose](https://github.com)
		- [2\.11、浏览器登录 neo4j](https://github.com)
* [二、Neo4j 初始配置](https://github.com)
	+ [1、清空 Neo4j 数据库](https://github.com)
* [三、PyCharm 项目安装必备库](https://github.com)
	+ [1、py2neo 库](https://github.com)
	+ [2、pymongo 库](https://github.com)
	+ [3、lxml 库](https://github.com)
* [四、python 连接 Neo4j](https://github.com)
	+ [1、浏览器 browser 查看Neo4j 连接状态](https://github.com)
	+ [2、修改源文件中 Graph 连接格式](https://github.com)
* [五、PyCharm 导入医疗知识图谱](https://github.com)
	+ [1、读取文件](https://github.com)
	+ [2、建立节点](https://github.com)
	+ [3、创建知识图谱中心疾病的节点](https://github.com)
	+ [4、创建知识图谱实体节点类型schema](https://github.com)
	+ [5、创建实体关系边](https://github.com)
	+ [6、创建实体关联边](https://github.com)
	+ [7、导出数据](https://github.com)
	+ [8、程序主入口](https://github.com)
		- [8\.1、UnicodeDecodeError: 'gbk' codec can't decode byte 0xaf in position 81: illegal multibyte sequence](https://github.com)
		- [8\.2、修改代码：for data in open(self.data\_path):](https://github.com)
	+ [9、运行结果](https://github.com)
	+ [10、优化导入数据时间](https://github.com)
* [六、PyCharm 实现问答系统](https://github.com)
	+ [1、问句类型分类脚本](https://github.com)
	+ [2、问句解析脚本](https://github.com)
	+ [3、问答程序脚本](https://github.com)
	+ [4、问答系统实现](https://github.com)
		- [4\.1、模型初始化](https://github.com)
		- [4\.2、问答主函数](https://github.com)
		- [4\.3、运行主入口](https://github.com)
		- [4\.4、运行结果](https://github.com)


> **说在前面：参考刘焕勇老师在 Github 上开源的项目**
> 
> 
> GitHub地址：[基于知识图谱的医疗问答系统](https://github.com)


# 一、搭建 Neo4j 图数据库


## 1、方式选择


* windows 使用 Neo4j Desktop （2024\-12\-09开始 Neo4j desktop 无法打开表现为三个/四个僵尸进程，查看本地日志会发现\[403]无法获取到https://dist.neo4j.org/neo4j\-desktop/win/latest.yml这个路径的资源。解决方案：断网打开 Neo4j Desktop / [Neo4j Desktop 1\.5\.8 Launches Zombie Processes Only \- Neo4j Graph Platform / Desktop \- Neo4j Online Community](https://github.com)）
* 云环境 dockerfile \+ docker\-compose （部署构建简单易懂无需专注 jdk 版本，优先考虑）
* 最终理想化：kubernetes 部署 （符合主流技术导向，虽说部署较复杂且多坑但是企业级以及行业主导地位等因素使用 k8s 部署还是最佳实践）



> 首次部署优先采用 dockerfile \+ docker\-compose


## 2、Dockerfile\+docker\-compose部署neo4j容器


### 2\.1、更新 yum 镜像源


bash
```
rm -rf /etc/yum.repos.d/*
wget -O /etc/yum.repos.d/centos7.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 2\.2、安装 docker\-ce 社区版


bash
```
yum install -y docker-ce
```

### 2\.3、配置镜像加速


bash
```
cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": [
    "https://dockerhub.icu",
    "https://hub.rat.dev",
    "https://docker.wanpeng.top",
    "https://doublezonline.cloud",
    "https://docker.mrxn.net",
    "https://docker.anyhub.us.kg",
    "https://dislabaiot.xyz",
    "https://docker.fxxk.dedyn.io"
  ]
}
EOF

systemctl daemon-reload && systemctl restart docker && systemctl enable docker
```

### 2\.4、安装 Docker Compose


[Releases · docker/compose](https://github.com)


#### 2\.4\.1、下载 Docker Compose 二进制包


bash
```
curl -L "https://github.com/docker/compose/releases/download/v2.5.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

* `-L`: 是`curl`的一个选项，表示跟随重定向。如果下载链接是重定向的，这个选项会让`curl`自动跟踪到最后的目标地址。
* `"https://github.com/docker/compose/releases/download/v2.5.1/docker-compose-$(uname -s)-$(uname -m)"`: 这是Docker Compose的下载URL，其中`v2.5.1`指定了要下载的Docker Compose版本号。`$(uname -s)` 和 `$(uname -m)` 是shell命令，分别返回当前系统的类型（如`Linux`）和机器的硬件架构（如`x86_64`），这样可以确保下载与当前系统架构相匹配的Docker Compose二进制文件。
* `-o /usr/local/bin/docker-compose`: `-o` 或 `--output` 指定了下载文件的保存位置及名称。这里，文件会被保存为 `/usr/local/bin/docker-compose`，这是Docker Compose常见的安装路径，将其放在此处可以使其在PATH环境变量中，从而可以直接在命令行中通过`docker-compose`命令调用。


#### 2\.4\.2、设置可执行权限


bash
```
chmod +x /usr/local/bin/docker-compose
```

#### 2\.4\.3、查看版本


bash
```
docker-compose -v
```

### 2\.5、创建目录结构


bash
```
mkdir -p neo4j-docker/{conf,data,import,logs} && touch neo4j-docker/conf/neo4j.conf

chown -R neo4j:neo4j ./{conf,data,import,logs}

chmod 755 ./{conf,data,logs,import}

tree -L 2 neo4j-docker
neo4j-docker
├── conf
│   └── neo4j.conf
├── data
├── import
└── logs
```

### 2\.6、编写neo4j.conf配置文件


bash
```
cat > /root/neo4j-docker/conf/neo4j.conf <<  EOF
server.directories.import=/var/lib/neo4j/import
server.memory.pagecache.size=512M

server.default_listen_address=0.0.0.0
dbms.security.allow_csv_import_from_file_urls=true
server.directories.logs=/logs
EOF
```

### 2\.7、编写 dockerfile 文件


dockerfile
```
cat > /root/neo4j-docker/Dockerfile << EOF
# 使用官方 Neo4j 最新版本镜像作为基础镜像
FROM neo4j:latest

# 设置环境变量，仅用于配置 Neo4j 认证
ENV NEO4J_AUTH=neo4j/neo4jpassword

# 拷贝本地的配置文件到容器中
COPY ./conf/neo4j.conf /var/lib/neo4j/conf/

# 定义容器启动时执行的命令
CMD ["neo4j"]
EOF
```

### 2\.8、构建ne4j容器镜像


bash
```
# 命令位置需要与Dockerfile位置同级
docker build -t my_neo4j:v1 .
```

[![](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114012599-1365709347.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114012599-1365709347.png)


### 2\.9、编写docker\-compose.yaml文件



> 有坑：neo4j 5\.x 版本所需密码位数需要在 8 位以上


yaml
```
version: '3'
services:
  neo4j:
    build: .
    image: my_neo4j:v1
    container_name: neo4j_container
    restart: always
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      - NEO4J_AUTH=neo4j/neo4jpassword
    volumes:
      - ./data:/data
      - ./logs:/logs
      - ./import:/var/lib/neo4j/import
      - ./conf:/var/lib/neo4j/conf
    command: ["neo4j"]
```

### 2\.10、运行docker\-compose


bash
```
docker-compose -f docker-compose.yaml up -d
```

### 2\.11、浏览器登录 neo4j


bash
```
http://192.168.112.30:7474

# 输入用户名:neo4j
# 输入密码:neo4jpassword
```

# 二、Neo4j 初始配置


## 1、清空 Neo4j 数据库


cypher
```
MATCH (n) DETACH DELETE n
```

[![](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114012273-1457313316.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114012273-1457313316.png)


# 三、PyCharm 项目安装必备库


## 1、py2neo 库


python
```
pip install py2neo
```

* **简化 Neo4j 连接和查询**


	+ **连接到 Neo4j**：`py2neo` 提供了简单易用的接口来连接到 Neo4j 数据库，支持 HTTP 和 Bolt 协议。
	+ **执行 Cypher 查询**：`py2neo` 允许你直接执行 **Cypher** 查询（Neo4j 的图查询语言），并以 Python 对象的形式返回结果。
* **创建和管理图数据**


	+ **创建节点和关系**：`py2neo` 提供了高级抽象，允许你像操作 Python 对象一样创建和管理 Neo4j 中的节点和关系。你可以使用 `Node` 和 `Relationship` 类来表示图中的实体，并将它们保存到数据库中。
	+ **批量操作**：`py2neo` 支持批量创建节点和关系，提高性能，减少网络往返次数。


## 2、pymongo 库


python
```
pip install pymongo
```

* 用于连接和操作 MongoDB 数据库，读取、处理并重新插入医疗数据。
* 提供了高效的 CRUD 操作，支持批量数据处理。


## 3、lxml 库


python
```
pip install lxml
```

* 用于解析存储在 MongoDB 中的 HTML 文档，提取有用的医疗检查信息（如疾病名称、描述等）。
* 通过 XPath 提取数据，并进行必要的清理和格式化。


# 四、python 连接 Neo4j


## 1、浏览器 browser 查看Neo4j 连接状态


cypher
```
:server status
```

[![](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114011965-1963768846.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114011965-1963768846.png):[wgetcloud加速器官网下载](https://longdu.org)



> 记住 URL （不是传统意义上的 http://，以及默认的端口号7474）


## 2、修改源文件中 Graph 连接格式


python
```
import os
import json
from py2neo import Graph,Node

class MedicalGraph:
    def __init__(self):
        cur_dir = '/'.join(os.path.abspath(__file__).split('/')[:-1])
        self.data_path = os.path.join(cur_dir, 'data/medical.json')
        self.g = Graph("neo4j://192.168.112.30:7687", auth=("neo4j", "neo4jpassword"))
```


> `build_medicalgraph.py` 和 `answer_search.py` 两个原文件中的 `self.g = Graph()` 的连接格式都更改为上述代码中的格式。


# 五、PyCharm 导入医疗知识图谱


## 1、读取文件


python
```
# 读取文件
    def read_nodes(self):
        # 共７类节点
        drugs = [] # 药品
        foods = [] #　食物
        checks = [] # 检查
        departments = [] #科室
        producers = [] #药品大类
        diseases = [] #疾病
        symptoms = []#症状

        disease_infos = []#疾病信息

        # 构建节点实体关系
        rels_department = [] #　科室－科室关系
        rels_noteat = [] # 疾病－忌吃食物关系
        rels_doeat = [] # 疾病－宜吃食物关系
        rels_recommandeat = [] # 疾病－推荐吃食物关系
        rels_commonddrug = [] # 疾病－通用药品关系
        rels_recommanddrug = [] # 疾病－热门药品关系
        rels_check = [] # 疾病－检查关系
        rels_drug_producer = [] # 厂商－药物关系

        rels_symptom = [] #疾病症状关系
        rels_acompany = [] # 疾病并发关系
        rels_category = [] #　疾病与科室之间的关系


        count = 0
        for data in open(self.data_path, encoding='utf8', mode='r'):
            disease_dict = {}
            count += 1
            print(count)
            data_json = json.loads(data)
            disease = data_json['name']
            disease_dict['name'] = disease
            diseases.append(disease)
            disease_dict['desc'] = ''
            disease_dict['prevent'] = ''
            disease_dict['cause'] = ''
            disease_dict['easy_get'] = ''
            disease_dict['cure_department'] = ''
            disease_dict['cure_way'] = ''
            disease_dict['cure_lasttime'] = ''
            disease_dict['symptom'] = ''
            disease_dict['cured_prob'] = ''

            if 'symptom' in data_json:
                symptoms += data_json['symptom']
                for symptom in data_json['symptom']:
                    rels_symptom.append([disease, symptom])

            if 'acompany' in data_json:
                for acompany in data_json['acompany']:
                    rels_acompany.append([disease, acompany])

            if 'desc' in data_json:
                disease_dict['desc'] = data_json['desc']

            if 'prevent' in data_json:
                disease_dict['prevent'] = data_json['prevent']

            if 'cause' in data_json:
                disease_dict['cause'] = data_json['cause']

            if 'get_prob' in data_json:
                disease_dict['get_prob'] = data_json['get_prob']

            if 'easy_get' in data_json:
                disease_dict['easy_get'] = data_json['easy_get']

            if 'cure_department' in data_json:
                cure_department = data_json['cure_department']
                if len(cure_department) == 1:
                     rels_category.append([disease, cure_department[0]])
                if len(cure_department) == 2:
                    big = cure_department[0]
                    small = cure_department[1]
                    rels_department.append([small, big])
                    rels_category.append([disease, small])

                disease_dict['cure_department'] = cure_department
                departments += cure_department

            if 'cure_way' in data_json:
                disease_dict['cure_way'] = data_json['cure_way']

            if  'cure_lasttime' in data_json:
                disease_dict['cure_lasttime'] = data_json['cure_lasttime']

            if 'cured_prob' in data_json:
                disease_dict['cured_prob'] = data_json['cured_prob']

            if 'common_drug' in data_json:
                common_drug = data_json['common_drug']
                for drug in common_drug:
                    rels_commonddrug.append([disease, drug])
                drugs += common_drug

            if 'recommand_drug' in data_json:
                recommand_drug = data_json['recommand_drug']
                drugs += recommand_drug
                for drug in recommand_drug:
                    rels_recommanddrug.append([disease, drug])

            if 'not_eat' in data_json:
                not_eat = data_json['not_eat']
                for _not in not_eat:
                    rels_noteat.append([disease, _not])

                foods += not_eat
                do_eat = data_json['do_eat']
                for _do in do_eat:
                    rels_doeat.append([disease, _do])

                foods += do_eat
                recommand_eat = data_json['recommand_eat']

                for _recommand in recommand_eat:
                    rels_recommandeat.append([disease, _recommand])
                foods += recommand_eat

            if 'check' in data_json:
                check = data_json['check']
                for _check in check:
                    rels_check.append([disease, _check])
                checks += check
            if 'drug_detail' in data_json:
                drug_detail = data_json['drug_detail']
                producer = [i.split('(')[0] for i in drug_detail]
                rels_drug_producer += [[i.split('(')[0], i.split('(')[-1].replace(')', '')] for i in drug_detail]
                producers += producer
            disease_infos.append(disease_dict)
        return set(drugs), set(foods), set(checks), set(departments), set(producers), set(symptoms), set(diseases), disease_infos,\
               rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug,\
               rels_symptom, rels_acompany, rels_category
```

## 2、建立节点


python
```
# 建立节点
    def create_node(self, label, nodes):
        count = 0
        for node_name in nodes:
            node = Node(label, name=node_name)
            self.g.create(node)
            count += 1
            print(count, len(nodes))
        return
```

## 3、创建知识图谱中心疾病的节点


python
```
# 创建知识图谱中心疾病的节点
    def create_diseases_nodes(self, disease_infos):
        count = 0
        for disease_dict in disease_infos:
            node = Node("Disease", name=disease_dict['name'], desc=disease_dict['desc'],
                        prevent=disease_dict['prevent'] ,cause=disease_dict['cause'],
                        easy_get=disease_dict['easy_get'],cure_lasttime=disease_dict['cure_lasttime'],
                        cure_department=disease_dict['cure_department']
                        ,cure_way=disease_dict['cure_way'] , cured_prob=disease_dict['cured_prob'])
            self.g.create(node)
            count += 1
            print(count)
        return
```

## 4、创建知识图谱实体节点类型schema


python
```
# 创建知识图谱实体节点类型schema
    def create_graphnodes(self):
        Drugs, Foods, Checks, Departments, Producers, Symptoms, Diseases, disease_infos,rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug,rels_symptom, rels_acompany, rels_category = self.read_nodes()
        self.create_diseases_nodes(disease_infos)
        self.create_node('Drug', Drugs)
        print(len(Drugs))
        self.create_node('Food', Foods)
        print(len(Foods))
        self.create_node('Check', Checks)
        print(len(Checks))
        self.create_node('Department', Departments)
        print(len(Departments))
        self.create_node('Producer', Producers)
        print(len(Producers))
        self.create_node('Symptom', Symptoms)
        return
```

## 5、创建实体关系边


python
```
# 创建实体关系边
    def create_graphrels(self):
        Drugs, Foods, Checks, Departments, Producers, Symptoms, Diseases, disease_infos, rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug,rels_symptom, rels_acompany, rels_category = self.read_nodes()
        self.create_relationship('Disease', 'Food', rels_recommandeat, 'recommand_eat', '推荐食谱')
        self.create_relationship('Disease', 'Food', rels_noteat, 'no_eat', '忌吃')
        self.create_relationship('Disease', 'Food', rels_doeat, 'do_eat', '宜吃')
        self.create_relationship('Department', 'Department', rels_department, 'belongs_to', '属于')
        self.create_relationship('Disease', 'Drug', rels_commonddrug, 'common_drug', '常用药品')
        self.create_relationship('Producer', 'Drug', rels_drug_producer, 'drugs_of', '生产药品')
        self.create_relationship('Disease', 'Drug', rels_recommanddrug, 'recommand_drug', '好评药品')
        self.create_relationship('Disease', 'Check', rels_check, 'need_check', '诊断检查')
        self.create_relationship('Disease', 'Symptom', rels_symptom, 'has_symptom', '症状')
        self.create_relationship('Disease', 'Disease', rels_acompany, 'acompany_with', '并发症')
        self.create_relationship('Disease', 'Department', rels_category, 'belongs_to', '所属科室')
```

## 6、创建实体关联边


python
```
# 创建实体关联边
    def create_relationship(self, start_node, end_node, edges, rel_type, rel_name):
        count = 0
        # 去重处理
        set_edges = []
        for edge in edges:
            set_edges.append('###'.join(edge))
        all = len(set(set_edges))
        for edge in set(set_edges):
            edge = edge.split('###')
            p = edge[0]
            q = edge[1]
            query = "match(p:%s),(q:%s) where p.name='%s'and q.name='%s' create (p)-[rel:%s{name:'%s'}]->(q)" % (
                start_node, end_node, p, q, rel_type, rel_name)
            try:
                self.g.run(query)
                count += 1
                print(rel_type, count, all)
            except Exception as e:
                print(e)
        return
```

## 7、导出数据


python
```
# 导出数据
    def export_data(self):
        Drugs, Foods, Checks, Departments, Producers, Symptoms, Diseases, disease_infos, rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug, rels_symptom, rels_acompany, rels_category = self.read_nodes()
        f_drug = open('drug.txt', 'w+')
        f_food = open('food.txt', 'w+')
        f_check = open('check.txt', 'w+')
        f_department = open('department.txt', 'w+')
        f_producer = open('producer.txt', 'w+')
        f_symptom = open('symptoms.txt', 'w+')
        f_disease = open('disease.txt', 'w+')

        f_drug.write('\n'.join(list(Drugs)))
        f_food.write('\n'.join(list(Foods)))
        f_check.write('\n'.join(list(Checks)))
        f_department.write('\n'.join(list(Departments)))
        f_producer.write('\n'.join(list(Producers)))
        f_symptom.write('\n'.join(list(Symptoms)))
        f_disease.write('\n'.join(list(Diseases)))

        f_drug.close()
        f_food.close()
        f_check.close()
        f_department.close()
        f_producer.close()
        f_symptom.close()
        f_disease.close()

        return
```

## 8、程序主入口


python
```
if __name__ == '__main__':
    handler = MedicalGraph()
    print("step1:导入图谱节点中")
    handler.create_graphnodes()
    print("step2:导入图谱边中")      
    handler.create_graphrels()
```

bash
```
# 创建知识节点和边（nodes + rels）
# handler.create_graphnodes()
# handler.create_graphrels()
快捷键：Ctrl + Shift + F10
```

### 8\.1、UnicodeDecodeError: 'gbk' codec can't decode byte 0xaf in position 81: illegal multibyte sequence



> 直接运行会报错：UnicodeDecodeError: 'gbk' codec can't decode byte 0xaf in position 81: illegal multibyte sequence


### 8\.2、修改代码：for data in open(self.data\_path):


bash
```
for data in open(self.data_path, encoding='utf8', mode='r'):
```

* 需要确保文件的编码格式为 utf8
* 打开文件模式为只读模式


## 9、运行结果


[![](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114011702-708543022.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114011702-708543022.png)


## 10、优化导入数据时间


python
```
import concurrent
import concurrent.futures
import json
import multiprocessing
import os

from py2neo import Graph, Node, Subgraph
from tqdm import tqdm


class MedicalGraph:
    def __init__(self):
        pass

    def clear(self):
        self.g.run("MATCH (n) DETACH DELETE n")

    '''读取文件'''

    def read_nodes(self):
        # 共７类节点
        drugs = []  # 药品
        foods = []  # 食物
        checks = []  # 检查
        departments = []  # 科室
        producers = []  # 药品大类
        diseases = []  # 疾病
        symptoms = []  # 症状

        disease_infos = []  # 疾病信息

        # 构建节点实体关系
        rels_department = []  # 科室－科室关系
        rels_noteat = []  # 疾病－忌吃食物关系
        rels_doeat = []  # 疾病－宜吃食物关系
        rels_recommandeat = []  # 疾病－推荐吃食物关系
        rels_commonddrug = []  # 疾病－通用药品关系
        rels_recommanddrug = []  # 疾病－热门药品关系
        rels_check = []  # 疾病－检查关系
        rels_drug_producer = []  # 厂商－药物关系

        rels_symptom = []  # 疾病症状关系
        rels_acompany = []  # 疾病并发关系
        rels_category = []  # 疾病与科室之间的关系

        for data in open(self.data_path):
            disease_dict = {}
            data_json = json.loads(data)
            disease = data_json['name']
            disease_dict['name'] = disease
            diseases.append(disease)
            disease_dict['desc'] = ''
            disease_dict['prevent'] = ''
            disease_dict['cause'] = ''
            disease_dict['easy_get'] = ''
            disease_dict['cure_department'] = ''
            disease_dict['cure_way'] = ''
            disease_dict['cure_lasttime'] = ''
            disease_dict['symptom'] = ''
            disease_dict['cured_prob'] = ''

            if 'symptom' in data_json:
                symptoms += data_json['symptom']
                for symptom in data_json['symptom']:
                    rels_symptom.append([disease, symptom])

            if 'acompany' in data_json:
                for acompany in data_json['acompany']:
                    rels_acompany.append([disease, acompany])

            if 'desc' in data_json:
                disease_dict['desc'] = data_json['desc']

            if 'prevent' in data_json:
                disease_dict['prevent'] = data_json['prevent']

            if 'cause' in data_json:
                disease_dict['cause'] = data_json['cause']

            if 'get_prob' in data_json:
                disease_dict['get_prob'] = data_json['get_prob']

            if 'easy_get' in data_json:
                disease_dict['easy_get'] = data_json['easy_get']

            if 'cure_department' in data_json:
                cure_department = data_json['cure_department']
                if len(cure_department) == 1:
                    rels_category.append([disease, cure_department[0]])
                if len(cure_department) == 2:
                    big = cure_department[0]
                    small = cure_department[1]
                    rels_department.append([small, big])
                    rels_category.append([disease, small])

                disease_dict['cure_department'] = cure_department
                departments += cure_department

            if 'cure_way' in data_json:
                disease_dict['cure_way'] = data_json['cure_way']

            if 'cure_lasttime' in data_json:
                disease_dict['cure_lasttime'] = data_json['cure_lasttime']

            if 'cured_prob' in data_json:
                disease_dict['cured_prob'] = data_json['cured_prob']

            if 'common_drug' in data_json:
                common_drug = data_json['common_drug']
                for drug in common_drug:
                    rels_commonddrug.append([disease, drug])
                drugs += common_drug

            if 'recommand_drug' in data_json:
                recommand_drug = data_json['recommand_drug']
                drugs += recommand_drug
                for drug in recommand_drug:
                    rels_recommanddrug.append([disease, drug])

            if 'not_eat' in data_json:
                not_eat = data_json['not_eat']
                for _not in not_eat:
                    rels_noteat.append([disease, _not])

                foods += not_eat
                do_eat = data_json['do_eat']
                for _do in do_eat:
                    rels_doeat.append([disease, _do])

                foods += do_eat
                recommand_eat = data_json['recommand_eat']

                for _recommand in recommand_eat:
                    rels_recommandeat.append([disease, _recommand])
                foods += recommand_eat

            if 'check' in data_json:
                check = data_json['check']
                for _check in check:
                    rels_check.append([disease, _check])
                checks += check
            if 'drug_detail' in data_json:
                drug_detail = data_json['drug_detail']
                producer = [i.split('(')[0] for i in drug_detail]
                rels_drug_producer += [[i.split('(')[0], i.split('(')[-1].replace(')', '')] for i in drug_detail]
                producers += producer
            disease_infos.append(disease_dict)
        return set(drugs), set(foods), set(checks), set(departments), set(producers), set(symptoms), set(diseases), disease_infos, \
            rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug, \
            rels_symptom, rels_acompany, rels_category

    '''建立节点'''

    def create_node(self, label, nodes):
        batch_size = 1000
        batches = [list(nodes)[i:i + batch_size] for i in range(0, len(nodes), batch_size)]
        for batch in tqdm(batches, desc=f"Creating {label} Nodes", unit="batch"):
            batch_nodes = [Node(label, name=node_name) for node_name in batch]
            self.g.create(Subgraph(batch_nodes))

    '''创建知识图谱中心疾病的节点'''

    def create_diseases_nodes(self, disease_infos):
        batch_size = 1000
        batches = [disease_infos[i:i + batch_size] for i in range(0, len(disease_infos), batch_size)]
        for batch in tqdm(batches, desc="Importing Disease Nodes", unit="batch"):
            batch_nodes = [
                Node("Disease", name=disease_dict['name'], desc=disease_dict['desc'],
                     prevent=disease_dict['prevent'], cause=disease_dict['cause'],
                     easy_get=disease_dict['easy_get'], cure_lasttime=disease_dict['cure_lasttime'],
                     cure_department=disease_dict['cure_department'], cure_way=disease_dict['cure_way'],
                     cured_prob=disease_dict['cured_prob']) for disease_dict in batch
            ]
            self.g.create(Subgraph(batch_nodes))

    '''创建知识图谱实体节点类型schema'''

    def create_graphnodes(self):
        Drugs, Foods, Checks, Departments, Producers, Symptoms, Diseases, disease_infos, rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug, rels_symptom, rels_acompany, rels_category = self.read_nodes()
        self.create_diseases_nodes(disease_infos)
        self.create_node('Drug', Drugs)
        self.create_node('Food', Foods)
        self.create_node('Check', Checks)
        self.create_node('Department', Departments)
        self.create_node('Producer', Producers)
        self.create_node('Symptom', Symptoms)

    '''创建实体关系边'''

    def create_graphrels(self):
        Drugs, Foods, Checks, Departments, Producers, Symptoms, Diseases, disease_infos, rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug, rels_symptom, rels_acompany, rels_category = self.read_nodes()
        self.create_relationship('Disease', 'Food', rels_recommandeat, 'recommand_eat', '推荐食谱')
        self.create_relationship('Disease', 'Food', rels_noteat, 'no_eat', '忌吃')
        self.create_relationship('Disease', 'Food', rels_doeat, 'do_eat', '宜吃')
        self.create_relationship('Department', 'Department', rels_department, 'belongs_to', '属于')
        self.create_relationship('Disease', 'Drug', rels_commonddrug, 'common_drug', '常用药品')
        self.create_relationship('Producer', 'Drug', rels_drug_producer, 'drugs_of', '生产药品')
        self.create_relationship('Disease', 'Drug', rels_recommanddrug, 'recommand_drug', '好评药品')
        self.create_relationship('Disease', 'Check', rels_check, 'need_check', '诊断检查')
        self.create_relationship('Disease', 'Symptom', rels_symptom, 'has_symptom', '症状')
        self.create_relationship('Disease', 'Disease', rels_acompany, 'acompany_with', '并发症')
        self.create_relationship('Disease', 'Department', rels_category, 'belongs_to', '所属科室')

    '''创建实体关联边'''

    def create_relationship(self, start_node, end_node, edges, rel_type, rel_name):
        batch_size = 10000
        set_edges = set(['###'.join(edge) for edge in edges])
        batches = [list(set_edges)[i:i + batch_size] for i in range(0, len(set_edges), batch_size)]
        executor = concurrent.futures.ThreadPoolExecutor(max_workers=min(multiprocessing.cpu_count(), 4))
        tasks = [
            lambda: (
                tx := self.g.begin(),
                [
                    tx.run(
                        f"MATCH (p:{start_node}), (q:{end_node}) "
                        f"WHERE p.name='{p}' AND q.name='{q}' "
                        f"CREATE (p)-[rel:{rel_type} {{name:'{rel_name}'}}]->(q)"
                    ) for edge in batch for p, q in [edge.split('###')]
                ],
                self.g.commit(tx)
            ) for batch in tqdm(batches, desc=f"Creating {rel_type} Relationships", unit="batch")
        ]
        executor.map(lambda task: task(), tasks)
        executor.shutdown()

    '''导出数据'''

    def export_data(self):
        Drugs, Foods, Checks, Departments, Producers, Symptoms, Diseases, disease_infos, rels_check, rels_recommandeat, rels_noteat, rels_doeat, rels_department, rels_commonddrug, rels_drug_producer, rels_recommanddrug, rels_symptom, rels_acompany, rels_category = self.read_nodes()
        f_drug = open('drug.txt', 'w+')
        f_food = open('food.txt', 'w+')
        f_check = open('check.txt', 'w+')
        f_department = open('department.txt', 'w+')
        f_producer = open('producer.txt', 'w+')
        f_symptom = open('symptoms.txt', 'w+')
        f_disease = open('disease.txt', 'w+')

        f_drug.write('\n'.join(list(Drugs)))
        f_food.write('\n'.join(list(Foods)))
        f_check.write('\n'.join(list(Checks)))
        f_department.write('\n'.join(list(Departments)))
        f_producer.write('\n'.join(list(Producers)))
        f_symptom.write('\n'.join(list(Symptoms)))
        f_disease.write('\n'.join(list(Diseases)))

        f_drug.close()
        f_food.close()
        f_check.close()
        f_department.close()
        f_producer.close()
        f_symptom.close()
        f_disease.close()


if __name__ == '__main__':
    handler = MedicalGraph()
    handler.clear()
    print("step1:导入图谱节点中")
    handler.create_graphnodes()
    print("step2:导入图谱边中")
    handler.create_graphrels()
```

# 六、PyCharm 实现问答系统


## 1、问句类型分类脚本



> 这里 **加载多个特征词列表** 处需要保证文件编码格式为 **utf8**
> 
> 
> 即添加内容：encoding\='utf8'


python
```
import os
import ahocorasick

class QuestionClassifier:
    def __init__(self):
        cur_dir = '/'.join(os.path.abspath(__file__).split('/')[:-1])
        #　特征词路径
        self.disease_path = os.path.join(cur_dir, 'dict/disease.txt')
        self.department_path = os.path.join(cur_dir, 'dict/department.txt')
        self.check_path = os.path.join(cur_dir, 'dict/check.txt')
        self.drug_path = os.path.join(cur_dir, 'dict/drug.txt')
        self.food_path = os.path.join(cur_dir, 'dict/food.txt')
        self.producer_path = os.path.join(cur_dir, 'dict/producer.txt')
        self.symptom_path = os.path.join(cur_dir, 'dict/symptom.txt')
        self.deny_path = os.path.join(cur_dir, 'dict/deny.txt')
        # 加载特征词
        self.disease_wds= [i.strip() for i in open(self.disease_path,encoding='utf8') if i.strip()]
        self.department_wds= [i.strip() for i in open(self.department_path,encoding='utf8') if i.strip()]
        self.check_wds= [i.strip() for i in open(self.check_path,encoding='utf8') if i.strip()]
        self.drug_wds= [i.strip() for i in open(self.drug_path,encoding='utf8') if i.strip()]
        self.food_wds= [i.strip() for i in open(self.food_path,encoding='utf8') if i.strip()]
        self.producer_wds= [i.strip() for i in open(self.producer_path,encoding='utf8') if i.strip()]
        self.symptom_wds= [i.strip() for i in open(self.symptom_path,encoding='utf8') if i.strip()]
        self.region_words = set(self.department_wds + self.disease_wds + self.check_wds + self.drug_wds + self.food_wds + self.producer_wds + self.symptom_wds)
        self.deny_words = [i.strip() for i in open(self.deny_path,encoding='utf8') if i.strip()]
        # 构造领域actree
        self.region_tree = self.build_actree(list(self.region_words))
        # 构建词典
        self.wdtype_dict = self.build_wdtype_dict()
        # 问句疑问词
        self.symptom_qwds = ['症状', '表征', '现象', '症候', '表现']
        self.cause_qwds = ['原因','成因', '为什么', '怎么会', '怎样才', '咋样才', '怎样会', '如何会', '为啥', '为何', '如何才会', '怎么才会', '会导致', '会造成']
        self.acompany_qwds = ['并发症', '并发', '一起发生', '一并发生', '一起出现', '一并出现', '一同发生', '一同出现', '伴随发生', '伴随', '共现']
        self.food_qwds = ['饮食', '饮用', '吃', '食', '伙食', '膳食', '喝', '菜' ,'忌口', '补品', '保健品', '食谱', '菜谱', '食用', '食物','补品']
        self.drug_qwds = ['药', '药品', '用药', '胶囊', '口服液', '炎片']
        self.prevent_qwds = ['预防', '防范', '抵制', '抵御', '防止','躲避','逃避','避开','免得','逃开','避开','避掉','躲开','躲掉','绕开',
                             '怎样才能不', '怎么才能不', '咋样才能不','咋才能不', '如何才能不',
                             '怎样才不', '怎么才不', '咋样才不','咋才不', '如何才不',
                             '怎样才可以不', '怎么才可以不', '咋样才可以不', '咋才可以不', '如何可以不',
                             '怎样才可不', '怎么才可不', '咋样才可不', '咋才可不', '如何可不']
        self.lasttime_qwds = ['周期', '多久', '多长时间', '多少时间', '几天', '几年', '多少天', '多少小时', '几个小时', '多少年']
        self.cureway_qwds = ['怎么治疗', '如何医治', '怎么医治', '怎么治', '怎么医', '如何治', '医治方式', '疗法', '咋治', '怎么办', '咋办', '咋治']
        self.cureprob_qwds = ['多大概率能治好', '多大几率能治好', '治好希望大么', '几率', '几成', '比例', '可能性', '能治', '可治', '可以治', '可以医']
        self.easyget_qwds = ['易感人群', '容易感染', '易发人群', '什么人', '哪些人', '感染', '染上', '得上']
        self.check_qwds = ['检查', '检查项目', '查出', '检查', '测出', '试出']
        self.belong_qwds = ['属于什么科', '属于', '什么科', '科室']
        self.cure_qwds = ['治疗什么', '治啥', '治疗啥', '医治啥', '治愈啥', '主治啥', '主治什么', '有什么用', '有何用', '用处', '用途',
                          '有什么好处', '有什么益处', '有何益处', '用来', '用来做啥', '用来作甚', '需要', '要']

        print('model init finished ......')

        return

    '''分类主函数'''
    def classify(self, question):
        data = {}
        medical_dict = self.check_medical(question)
        if not medical_dict:
            return {}
        data['args'] = medical_dict
        #收集问句当中所涉及到的实体类型
        types = []
        for type_ in medical_dict.values():
            types += type_
        question_type = 'others'

        question_types = []

        # 症状
        if self.check_words(self.symptom_qwds, question) and ('disease' in types):
            question_type = 'disease_symptom'
            question_types.append(question_type)

        if self.check_words(self.symptom_qwds, question) and ('symptom' in types):
            question_type = 'symptom_disease'
            question_types.append(question_type)

        # 原因
        if self.check_words(self.cause_qwds, question) and ('disease' in types):
            question_type = 'disease_cause'
            question_types.append(question_type)
        # 并发症
        if self.check_words(self.acompany_qwds, question) and ('disease' in types):
            question_type = 'disease_acompany'
            question_types.append(question_type)

        # 推荐食品
        if self.check_words(self.food_qwds, question) and 'disease' in types:
            deny_status = self.check_words(self.deny_words, question)
            if deny_status:
                question_type = 'disease_not_food'
            else:
                question_type = 'disease_do_food'
            question_types.append(question_type)

        #已知食物找疾病
        if self.check_words(self.food_qwds+self.cure_qwds, question) and 'food' in types:
            deny_status = self.check_words(self.deny_words, question)
            if deny_status:
                question_type = 'food_not_disease'
            else:
                question_type = 'food_do_disease'
            question_types.append(question_type)

        # 推荐药品
        if self.check_words(self.drug_qwds, question) and 'disease' in types:
            question_type = 'disease_drug'
            question_types.append(question_type)

        # 药品治啥病
        if self.check_words(self.cure_qwds, question) and 'drug' in types:
            question_type = 'drug_disease'
            question_types.append(question_type)

        # 疾病接受检查项目
        if self.check_words(self.check_qwds, question) and 'disease' in types:
            question_type = 'disease_check'
            question_types.append(question_type)

        # 已知检查项目查相应疾病
        if self.check_words(self.check_qwds+self.cure_qwds, question) and 'check' in types:
            question_type = 'check_disease'
            question_types.append(question_type)

        #　症状防御
        if self.check_words(self.prevent_qwds, question) and 'disease' in types:
            question_type = 'disease_prevent'
            question_types.append(question_type)

        # 疾病医疗周期
        if self.check_words(self.lasttime_qwds, question) and 'disease' in types:
            question_type = 'disease_lasttime'
            question_types.append(question_type)

        # 疾病治疗方式
        if self.check_words(self.cureway_qwds, question) and 'disease' in types:
            question_type = 'disease_cureway'
            question_types.append(question_type)

        # 疾病治愈可能性
        if self.check_words(self.cureprob_qwds, question) and 'disease' in types:
            question_type = 'disease_cureprob'
            question_types.append(question_type)

        # 疾病易感染人群
        if self.check_words(self.easyget_qwds, question) and 'disease' in types :
            question_type = 'disease_easyget'
            question_types.append(question_type)

        # 若没有查到相关的外部查询信息，那么则将该疾病的描述信息返回
        if question_types == [] and 'disease' in types:
            question_types = ['disease_desc']

        # 若没有查到相关的外部查询信息，那么则将该疾病的描述信息返回
        if question_types == [] and 'symptom' in types:
            question_types = ['symptom_disease']

        # 将多个分类结果进行合并处理，组装成一个字典
        data['question_types'] = question_types

        return data

    '''构造词对应的类型'''
    def build_wdtype_dict(self):
        wd_dict = dict()
        for wd in self.region_words:
            wd_dict[wd] = []
            if wd in self.disease_wds:
                wd_dict[wd].append('disease')
            if wd in self.department_wds:
                wd_dict[wd].append('department')
            if wd in self.check_wds:
                wd_dict[wd].append('check')
            if wd in self.drug_wds:
                wd_dict[wd].append('drug')
            if wd in self.food_wds:
                wd_dict[wd].append('food')
            if wd in self.symptom_wds:
                wd_dict[wd].append('symptom')
            if wd in self.producer_wds:
                wd_dict[wd].append('producer')
        return wd_dict

    '''构造actree，加速过滤'''
    def build_actree(self, wordlist):
        actree = ahocorasick.Automaton()
        for index, word in enumerate(wordlist):
            actree.add_word(word, (index, word))
        actree.make_automaton()
        return actree

    '''问句过滤'''
    def check_medical(self, question):
        region_wds = []
        for i in self.region_tree.iter(question):
            wd = i[1][1]
            region_wds.append(wd)
        stop_wds = []
        for wd1 in region_wds:
            for wd2 in region_wds:
                if wd1 in wd2 and wd1 != wd2:
                    stop_wds.append(wd1)
        final_wds = [i for i in region_wds if i not in stop_wds]
        final_dict = {i:self.wdtype_dict.get(i) for i in final_wds}

        return final_dict

    '''基于特征词进行分类'''
    def check_words(self, wds, sent):
        for wd in wds:
            if wd in sent:
                return True
        return False


if __name__ == '__main__':
    handler = QuestionClassifier()
    while 1:
        question = input('input an question:')
        data = handler.classify(question)
        print(data)
```

## 2、问句解析脚本


python
```
class QuestionPaser:

    '''构建实体节点'''
    def build_entitydict(self, args):
        entity_dict = {}
        for arg, types in args.items():
            for type in types:
                if type not in entity_dict:
                    entity_dict[type] = [arg]
                else:
                    entity_dict[type].append(arg)

        return entity_dict

    '''解析主函数'''
    def parser_main(self, res_classify):
        args = res_classify['args']
        entity_dict = self.build_entitydict(args)
        question_types = res_classify['question_types']
        sqls = []
        for question_type in question_types:
            sql_ = {}
            sql_['question_type'] = question_type
            sql = []
            if question_type == 'disease_symptom':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'symptom_disease':
                sql = self.sql_transfer(question_type, entity_dict.get('symptom'))

            elif question_type == 'disease_cause':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_acompany':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_not_food':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_do_food':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'food_not_disease':
                sql = self.sql_transfer(question_type, entity_dict.get('food'))

            elif question_type == 'food_do_disease':
                sql = self.sql_transfer(question_type, entity_dict.get('food'))

            elif question_type == 'disease_drug':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'drug_disease':
                sql = self.sql_transfer(question_type, entity_dict.get('drug'))

            elif question_type == 'disease_check':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'check_disease':
                sql = self.sql_transfer(question_type, entity_dict.get('check'))

            elif question_type == 'disease_prevent':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_lasttime':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_cureway':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_cureprob':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_easyget':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            elif question_type == 'disease_desc':
                sql = self.sql_transfer(question_type, entity_dict.get('disease'))

            if sql:
                sql_['sql'] = sql

                sqls.append(sql_)

        return sqls

    '''针对不同的问题，分开进行处理'''
    def sql_transfer(self, question_type, entities):
        if not entities:
            return []

        # 查询语句
        sql = []
        # 查询疾病的原因
        if question_type == 'disease_cause':
            sql = ["MATCH (m:Disease) where m.name = '{0}' return m.name, m.cause".format(i) for i in entities]

        # 查询疾病的防御措施
        elif question_type == 'disease_prevent':
            sql = ["MATCH (m:Disease) where m.name = '{0}' return m.name, m.prevent".format(i) for i in entities]

        # 查询疾病的持续时间
        elif question_type == 'disease_lasttime':
            sql = ["MATCH (m:Disease) where m.name = '{0}' return m.name, m.cure_lasttime".format(i) for i in entities]

        # 查询疾病的治愈概率
        elif question_type == 'disease_cureprob':
            sql = ["MATCH (m:Disease) where m.name = '{0}' return m.name, m.cured_prob".format(i) for i in entities]

        # 查询疾病的治疗方式
        elif question_type == 'disease_cureway':
            sql = ["MATCH (m:Disease) where m.name = '{0}' return m.name, m.cure_way".format(i) for i in entities]

        # 查询疾病的易发人群
        elif question_type == 'disease_easyget':
            sql = ["MATCH (m:Disease) where m.name = '{0}' return m.name, m.easy_get".format(i) for i in entities]

        # 查询疾病的相关介绍
        elif question_type == 'disease_desc':
            sql = ["MATCH (m:Disease) where m.name = '{0}' return m.name, m.desc".format(i) for i in entities]

        # 查询疾病有哪些症状
        elif question_type == 'disease_symptom':
            sql = ["MATCH (m:Disease)-[r:has_symptom]->(n:Symptom) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]

        # 查询症状会导致哪些疾病
        elif question_type == 'symptom_disease':
            sql = ["MATCH (m:Disease)-[r:has_symptom]->(n:Symptom) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]

        # 查询疾病的并发症
        elif question_type == 'disease_acompany':
            sql1 = ["MATCH (m:Disease)-[r:acompany_with]->(n:Disease) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql2 = ["MATCH (m:Disease)-[r:acompany_with]->(n:Disease) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql = sql1 + sql2
        # 查询疾病的忌口
        elif question_type == 'disease_not_food':
            sql = ["MATCH (m:Disease)-[r:no_eat]->(n:Food) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]

        # 查询疾病建议吃的东西
        elif question_type == 'disease_do_food':
            sql1 = ["MATCH (m:Disease)-[r:do_eat]->(n:Food) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql2 = ["MATCH (m:Disease)-[r:recommand_eat]->(n:Food) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql = sql1 + sql2

        # 已知忌口查疾病
        elif question_type == 'food_not_disease':
            sql = ["MATCH (m:Disease)-[r:no_eat]->(n:Food) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]

        # 已知推荐查疾病
        elif question_type == 'food_do_disease':
            sql1 = ["MATCH (m:Disease)-[r:do_eat]->(n:Food) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql2 = ["MATCH (m:Disease)-[r:recommand_eat]->(n:Food) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql = sql1 + sql2

        # 查询疾病常用药品－药品别名记得扩充
        elif question_type == 'disease_drug':
            sql1 = ["MATCH (m:Disease)-[r:common_drug]->(n:Drug) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql2 = ["MATCH (m:Disease)-[r:recommand_drug]->(n:Drug) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql = sql1 + sql2

        # 已知药品查询能够治疗的疾病
        elif question_type == 'drug_disease':
            sql1 = ["MATCH (m:Disease)-[r:common_drug]->(n:Drug) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql2 = ["MATCH (m:Disease)-[r:recommand_drug]->(n:Drug) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]
            sql = sql1 + sql2
        # 查询疾病应该进行的检查
        elif question_type == 'disease_check':
            sql = ["MATCH (m:Disease)-[r:need_check]->(n:Check) where m.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]

        # 已知检查查询疾病
        elif question_type == 'check_disease':
            sql = ["MATCH (m:Disease)-[r:need_check]->(n:Check) where n.name = '{0}' return m.name, r.name, n.name".format(i) for i in entities]

        return sql



if __name__ == '__main__':
    handler = QuestionPaser()
```

## 3、问答程序脚本


python
```
from py2neo import Graph

class AnswerSearcher:
    def __init__(self):
        self.g = Graph("neo4j://192.168.112.30:7687", auth=("neo4j", "neo4jpassword"))
        self.num_limit = 20

    '''执行cypher查询，并返回相应结果'''
    def search_main(self, sqls):
        final_answers = []
        for sql_ in sqls:
            question_type = sql_['question_type']
            queries = sql_['sql']
            answers = []
            for query in queries:
                ress = self.g.run(query).data()
                answers += ress
            final_answer = self.answer_prettify(question_type, answers)
            if final_answer:
                final_answers.append(final_answer)
        return final_answers

    '''根据对应的qustion_type，调用相应的回复模板'''
    def answer_prettify(self, question_type, answers):
        final_answer = []
        if not answers:
            return ''
        if question_type == 'disease_symptom':
            desc = [i['n.name'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}的症状包括：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'symptom_disease':
            desc = [i['m.name'] for i in answers]
            subject = answers[0]['n.name']
            final_answer = '症状{0}可能染上的疾病有：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_cause':
            desc = [i['m.cause'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}可能的成因有：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_prevent':
            desc = [i['m.prevent'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}的预防措施包括：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_lasttime':
            desc = [i['m.cure_lasttime'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}治疗可能持续的周期为：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_cureway':
            desc = [';'.join(i['m.cure_way']) for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}可以尝试如下治疗：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_cureprob':
            desc = [i['m.cured_prob'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}治愈的概率为（仅供参考）：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_easyget':
            desc = [i['m.easy_get'] for i in answers]
            subject = answers[0]['m.name']

            final_answer = '{0}的易感人群包括：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_desc':
            desc = [i['m.desc'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0},熟悉一下：{1}'.format(subject,  '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_acompany':
            desc1 = [i['n.name'] for i in answers]
            desc2 = [i['m.name'] for i in answers]
            subject = answers[0]['m.name']
            desc = [i for i in desc1 + desc2 if i != subject]
            final_answer = '{0}的症状包括：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_not_food':
            desc = [i['n.name'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}忌食的食物包括有：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_do_food':
            do_desc = [i['n.name'] for i in answers if i['r.name'] == '宜吃']
            recommand_desc = [i['n.name'] for i in answers if i['r.name'] == '推荐食谱']
            subject = answers[0]['m.name']
            final_answer = '{0}宜食的食物包括有：{1}\n推荐食谱包括有：{2}'.format(subject, ';'.join(list(set(do_desc))[:self.num_limit]), ';'.join(list(set(recommand_desc))[:self.num_limit]))

        elif question_type == 'food_not_disease':
            desc = [i['m.name'] for i in answers]
            subject = answers[0]['n.name']
            final_answer = '患有{0}的人最好不要吃{1}'.format('；'.join(list(set(desc))[:self.num_limit]), subject)

        elif question_type == 'food_do_disease':
            desc = [i['m.name'] for i in answers]
            subject = answers[0]['n.name']
            final_answer = '患有{0}的人建议多试试{1}'.format('；'.join(list(set(desc))[:self.num_limit]), subject)

        elif question_type == 'disease_drug':
            desc = [i['n.name'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}通常的使用的药品包括：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'drug_disease':
            desc = [i['m.name'] for i in answers]
            subject = answers[0]['n.name']
            final_answer = '{0}主治的疾病有{1},可以试试'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'disease_check':
            desc = [i['n.name'] for i in answers]
            subject = answers[0]['m.name']
            final_answer = '{0}通常可以通过以下方式检查出来：{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        elif question_type == 'check_disease':
            desc = [i['m.name'] for i in answers]
            subject = answers[0]['n.name']
            final_answer = '通常可以通过{0}检查出来的疾病有{1}'.format(subject, '；'.join(list(set(desc))[:self.num_limit]))

        return final_answer


if __name__ == '__main__':
    searcher = AnswerSearcher()
```

## 4、问答系统实现


### 4\.1、模型初始化


python
```
from answer_search import *
from question_classifier import *
from question_parser import *
 
 
class ChatBotGraph:
    def __init__(self):
        self.classifier = QuestionClassifier()
        self.parser = QuestionPaser()
        self.searcher = AnswerSearcher()
```

### 4\.2、问答主函数


python
```
    def chat_main(self, sent):
        answer = '您好，我是医药智能助理，希望可以帮到您。如果没答上来，可联系https://liuhuanyong.github.io/。祝您身体棒棒！'
        res_classify = self.classifier.classify(sent)
        if not res_classify:
            return answer
        res_sql = self.parser.parser_main(res_classify)
        final_answers = self.searcher.search_main(res_sql)
        if not final_answers:
            return answer
        else:
            return '\n'.join(final_answers)
```

### 4\.3、运行主入口



> 运行 chatbot\_graph.py 文件


python
```
if __name__ == '__main__':
    handler = ChatBotGraph()
    while 1:
        question = input('用户:')
        answer = handler.chat_main(question)
        print('医药智能助理:', answer)
```

### 4\.4、运行结果


[![](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114011109-903885731.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241218114011109-903885731.png)


  * [一、搭建 Neo4j 图数据库](#%E4%B8%80%E6%90%AD%E5%BB%BA-neo4j-%E5%9B%BE%E6%95%B0%E6%8D%AE%E5%BA%93)
* [1、方式选择](#tid-7rW7nw)
* [2、Dockerfile\+docker\-compose部署neo4j容器](#tid-7xAfBc)
* [2\.1、更新 yum 镜像源](#tid-YSrGjY)
* [2\.2、安装 docker\-ce 社区版](#tid-4bBA8E)
* [2\.3、配置镜像加速](#tid-dS7S7Q)
* [2\.4、安装 Docker Compose](#tid-awwMar)
* [2\.4\.1、下载 Docker Compose 二进制包](#tid-ipheN4)
* [2\.4\.2、设置可执行权限](#tid-ATPzWW)
* [2\.4\.3、查看版本](#tid-W52GZY)
* [2\.5、创建目录结构](#tid-MRGERp)
* [2\.6、编写neo4j.conf配置文件](#tid-tj6FJT)
* [2\.7、编写 dockerfile 文件](#tid-r8rjDX)
* [2\.8、构建ne4j容器镜像](#tid-xQn7PB)
* [2\.9、编写docker\-compose.yaml文件](#tid-2zck2P)
* [2\.10、运行docker\-compose](#tid-5587p7)
* [2\.11、浏览器登录 neo4j](#tid-b77mZy)
* [二、Neo4j 初始配置](#%E4%BA%8Cneo4j-%E5%88%9D%E5%A7%8B%E9%85%8D%E7%BD%AE)
* [1、清空 Neo4j 数据库](#tid-DpsiKE)
* [三、PyCharm 项目安装必备库](#%E4%B8%89pycharm-%E9%A1%B9%E7%9B%AE%E5%AE%89%E8%A3%85%E5%BF%85%E5%A4%87%E5%BA%93)
* [1、py2neo 库](#tid-GAiCEx)
* [2、pymongo 库](#tid-DacNtD)
* [3、lxml 库](#tid-2TBZNa)
* [四、python 连接 Neo4j](#%E5%9B%9Bpython-%E8%BF%9E%E6%8E%A5-neo4j)
* [1、浏览器 browser 查看Neo4j 连接状态](#tid-62Md2x)
* [2、修改源文件中 Graph 连接格式](#tid-Qe62G6)
* [五、PyCharm 导入医疗知识图谱](#%E4%BA%94pycharm-%E5%AF%BC%E5%85%A5%E5%8C%BB%E7%96%97%E7%9F%A5%E8%AF%86%E5%9B%BE%E8%B0%B1)
* [1、读取文件](#tid-GYPkYs)
* [2、建立节点](#tid-Fi7kTk)
* [3、创建知识图谱中心疾病的节点](#tid-px2dnc)
* [4、创建知识图谱实体节点类型schema](#tid-xNCPb8)
* [5、创建实体关系边](#tid-eime2b)
* [6、创建实体关联边](#tid-Y4xNAr)
* [7、导出数据](#tid-DY7amX)
* [8、程序主入口](#tid-HTe2aj)
* [8\.1、UnicodeDecodeError: 'gbk' codec can't decode byte 0xaf in position 81: illegal multibyte sequence](#tid-BkSTQD)
* [8\.2、修改代码：for data in open(self.data\_path):](#tid-rpXMhy)
* [9、运行结果](#tid-TcYKZr)
* [10、优化导入数据时间](#tid-GrQryM)
* [六、PyCharm 实现问答系统](#%E5%85%ADpycharm-%E5%AE%9E%E7%8E%B0%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F)
* [1、问句类型分类脚本](#tid-78aX75)
* [2、问句解析脚本](#tid-y8Xhzr)
* [3、问答程序脚本](#tid-zf3C3p)
* [4、问答系统实现](#tid-JNDzW2)
* [4\.1、模型初始化](#tid-eBZiQT)
* [4\.2、问答主函数](#tid-fAE7ah)
* [4\.3、运行主入口](#tid-ipzFii)
* [4\.4、运行结果](#tid-YysECH)

   \_\_EOF\_\_

   misakivv  - **本文链接：** [https://github.com/misakivv/p/18614532](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 除特殊说明外，转载请注明出处～\[知识共享署名\-相同方式共享 4\.0 国际许可协议]
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
