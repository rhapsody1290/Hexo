---
title: Java进行数据库备份、回复、删除

date: 2017-05-05 00:00:00

categories:
- 数据库

tags:
- 数据库

---

备份和删除使用Java的 Runtime 类的 exec 方法来执行命令行

	Process process = Runtime.getRuntime().exec(command);

但是命令行中无法识别管道符号 `<`，所以必须获得子进程的输入流和输出流写文件

清空数据库通过 `show tables` 获得数据库中所有表，然后通过 `truncate tableName` 命令来清空数据库，由于项目中使用 Hibernate 框架，所以利用 Hibernate 执行原生sql语句

## DatabaseBackupUtils

DatabaseBackupUtils 是数据库备份和还原的工具类，可以通用

	public class DatabaseBackupUtils {  
		
	    /** MySQL安装目录的Bin目录的绝对路径 */  
	    private String mysqlBinPath;  
	    /** 访问MySQL数据库的用户名 */  
	    private String username;  
	    /** 访问MySQL数据库的密码 */  
	    private String password;  
	    
	    public String getMysqlBinPath() {  
	        return mysqlBinPath;  
	    }  
	    public void setMysqlBinPath(String mysqlBinPath) {  
	        this.mysqlBinPath = mysqlBinPath;  
	    }  
	    public String getUsername() {  
	        return username;  
	    }  
	    public void setUsername(String username) {  
	        this.username = username;  
	    }  
	    public String getPassword() {  
	        return password;  
	    }  
	    public void setPassword(String password) {  
	        this.password = password;  
	    }  
	    public DatabaseBackupUtils(String mysqlBinPath, String username, String password) {  
	        if (!mysqlBinPath.endsWith(File.separator)) {  
	            mysqlBinPath = mysqlBinPath + File.separator;  
	        }  
	        this.mysqlBinPath = mysqlBinPath;  
	        this.username = username;  
	        this.password = password;  
	    }  
	    /** 
	     * 备份数据库 
	     *  
	     * @param output 
	     *            输出流 
	     * @param dbname 
	     *            要备份的数据库名 
	     */  
	    public void backup(OutputStream output, String dbname) {  
	    	
	    	/**
	    	 * 相当于在运行中输入：
	    	 * cmd /c D:/MySQL/bin\mysqldump -uroot -proot --set-charset=utf8 audit
	    	 * 注意：/c表示运行完命令后关闭进程，可以测试cmd /c ping localhost与cmd ping localhost的区别
	    	 */
	        String command = "cmd /c " + mysqlBinPath + "mysqldump -u" + username  
	                + " -p" + password + " --all-tablespaces --add-drop-database --set-charset=utf8 " + dbname;  
	        System.out.println(command);
	        PrintWriter writer = null;  
	        BufferedReader reader = null;  
	        try {  
	        	writer = new PrintWriter(new OutputStreamWriter(output, "utf8"));  
	            /**
	    		 * 执行mysqldump命令后会向子进程的缓存区写数据，以前在Dos命令行中通过重定向的方式将数据写入到文件中，
	    		 * 在这里必须从进程的读取数据，并写入sql文件，即生成备份文件 注：如果不对控制台信息进行读出，则会导致进程堵塞无法运行
	    		 * 把进程执行中的控制台输出信息写入.sql文件，即生成了备份文件。
	    		 */
	            Process process = Runtime.getRuntime().exec(command);  
	            //从缓存区中读数据
	            reader = new BufferedReader(new InputStreamReader(process.getInputStream(), "utf8"));  
	            String line = null;  
	            //边读边写
	            while ((line = reader.readLine()) != null) {  
	            	writer.println(line);  
	            }  
	            writer.flush();  
	        } catch (UnsupportedEncodingException e) {  
	            e.printStackTrace();  
	        } catch (IOException e) {  
	            e.printStackTrace();  
	        } finally {  
	            try {  
	                if (reader != null) {  
	                    reader.close();  
	                }  
	                if (writer != null) {  
	                	writer.close();  
	                }  
	            } catch (IOException e) {  
	                e.printStackTrace();  
	            }  
	        }  
	    }  
	    /** 
	     * 备份数据库，如果指定路径的文件不存在会自动生成 
	     *  
	     * @param dest 
	     *            备份文件的路径 
	     * @param dbname 
	     *            要备份的数据库 
	     */  
	    public void backup(String dest, String dbname) {  
	        try {  
	            OutputStream out = new FileOutputStream(dest);  
	            backup(out, dbname);  
	            System.out.println("备份成功");
	        } catch (FileNotFoundException e) {  
	            e.printStackTrace();  
	        }  
	    }  
	    /** 
	     * 恢复数据库 
	     *  
	     * @param input 
	     *            输入流 
	     * @param dbname 
	     *            数据库名 
	     */  
	    public void restore(InputStream input, String dbname) { 
	    	/**
	    	 * 在命令行中使用如下命令来实现恢复 cmd /c D:/MySQL/bin\mysql -uroot -proot test < test.sql
	    	 * 但是在Java中无法识别管道符号，于是向缓存区中写test.sql数据，来实现管道的效果
	    	 */
	        String command = "cmd /c " + mysqlBinPath + "mysql -u" + username  
	                + " -p" + password + " " + dbname;  
	        System.out.println(command);
	        try {  
	            Process process = Runtime.getRuntime().exec(command);  
	            OutputStream out = process.getOutputStream();  
	            String line = null;  
	            String outStr = null;  
	            StringBuffer sb = new StringBuffer("");  
	            BufferedReader br = new BufferedReader(new InputStreamReader(input,  
	                    "utf8"));  
	            while ((line = br.readLine()) != null) {  
	                sb.append(line + "\r\n");  
	            }  
	            outStr = sb.toString();
	            //System.out.println(outStr);
	            OutputStreamWriter writer = new OutputStreamWriter(out, "utf8");  
	            writer.write(outStr);  
	            writer.flush();  
	            out.close();  
	            br.close();  
	            writer.close();  
	        } catch (UnsupportedEncodingException e) {  
	            e.printStackTrace();  
	        } catch (IOException e) {  
	            e.printStackTrace();  
	        }  
	    }  
	    /** 
	     * 恢复数据库 
	     *  
	     * @param dest 
	     *            备份文件的路径 
	     * @param dbname 
	     *            数据库名 
	     * @throws FileNotFoundException 
	     */  
	    public void restore(String dest, String dbname) throws FileNotFoundException {  
	        InputStream input = new FileInputStream(dest);  
	        restore(input, dbname);  
	        System.out.println("恢复成功");
	    }  
	     
	    public static void main(String[] args) throws SQLException {  
	        String binPath = "D:/MySQL/bin";  
	        String userName = "root";  
	        String pwd = "root";  
	        DatabaseBackupUtils bak = new DatabaseBackupUtils(binPath, userName, pwd);  
	        //bak.backup("c:/users/asus/desktop/test.sql", "test");  
	        //bak.restore("c:/users/asus/desktop/test.sql", "test"); 
	    }  
	}  

## IDatabaseBackupDao、DatabaseBackupDaoImpl

DatabaseBackupDaoImpl 利用Hibernate执行sql语句清空数据库，分为两个步骤：

1、获得表名 
2、清空数据库

IDatabaseBackupDao

	public interface IDatabaseBackupDao extends ICommonDao<AuditAlert>{
		List<String> getTables();
		void clear(List<String> tables);
	}



DatabaseBackupDaoImpl

	@Component
	public class DatabaseBackupDaoImpl extends CommonDaoImpl<AuditAlert> implements
			IDatabaseBackupDao {
	
		@Override
		public List<String> getTables() {
	
			List<String> list = (List<String>) this.getHibernateTemplate().execute(
					new HibernateCallback() {
	
						@Override
						public Object doInHibernate(Session session)
								throws HibernateException, SQLException {
	
							List list = session
									.createSQLQuery(
											"select table_name from information_schema.TABLES where TABLE_SCHEMA = 'audit';")
									.list();
							return list;
						}
	
					});
			return list;
		}
	
		@Override
		public void clear(List<String> tables) {
			final List<String> f = tables;
			this.getHibernateTemplate().execute(new HibernateCallback() {
	
				Set<String> ignore = new HashSet<String>(Arrays.asList("remote_ip",
						"audit_user","audit_user_role","audit_role","audit_role_popedom",
						"audit_popedom"));
				
				@Override
				public Object doInHibernate(Session session)
						throws HibernateException, SQLException {
					for (String tableName : f) {
						if(ignore.contains(tableName)){
							continue;
						}
						session.createSQLQuery("truncate " + tableName).executeUpdate();
					}
					return null;
				}
			});
	
		}
	}

## IDataBackupService、DataBackupServiceImpl

IDataBackupService 定义了三个方法

1、备份数据库
2、恢复数据库
3、清空数据库

其中备份数据库通过 DatabaseBackupUtils 工具类实现，恢复数据库由于需要连接数据库，采用 IDatabaseBackupDao 类来实现

IDataBackupService

	public interface IDataBackupService {
		/**
		 * 备份数据库
		 * @throws IOException
		 */
		void doBackup() throws IOException;
		/**
		 * 恢复数据库
		 * @throws FileNotFoundException 
		 */
		void doRestore() throws FileNotFoundException;
		/**
		 * 清空数据库
		 */
		void doClear();
	}

DataBackupServiceImpl

	@Service
	public class DataBackupServiceImpl implements IDataBackupService {
	
		private static final String DB_USER = "root";
		private static final String DB_PWD = "root";
		private static final String DB_NAME = "audit";
		private static String MYSQL_BIN_PATH;
			
		static{
			BufferedReader br;
			try {
				br = new BufferedReader(new InputStreamReader(new FileInputStream("c:/AuditMysqlBinPath.txt")));
				MYSQL_BIN_PATH = br.readLine();
				if(br != null){
					br.close();
				}
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			
		}
		
		@Autowired
		IDatabaseBackupDao iDatabaseBackupDao;
		private static final DatabaseBackupUtils databaseBackupUtils = new DatabaseBackupUtils(MYSQL_BIN_PATH,DB_USER,DB_PWD);
	
		/**
		 * 备份数据库
		 */
		@Override
		public void doBackup() throws IOException {
			//如果audit.sql文件存在，可能是刚点击备份，所以不允许备份
			File file = new File("c:/audit.sql");
			if(file.exists()){
				throw new IllegalStateException();
			}
			databaseBackupUtils.backup("c:/audit.sql", "audit");
		}
	
		@Override
		public void doClear() {
			List<String> tables = iDatabaseBackupDao.getTables();
			System.out.println(tables);
			iDatabaseBackupDao.clear(tables);
		}
	
		@Override
		public void doRestore() throws FileNotFoundException {
			databaseBackupUtils.restore("c:/audit.sql", "audit");
		}
	}
