manager  redis和redisson管理类
test 测试类
util 锁工具类
uuid 生成uuid类



- 1. 不同进程之间是否真正能锁住
- 2. 集群正常的情况下， 出现问题是否可恢复
- 3. 性能 - 并发多少和响应时间

测试：用两个进程去跑进行测试


	1: 使用两个IDE对同一个key进行测试，结论：不同进程之间能够锁住！

	      测试过程：
	    （1）两个IDE同时启动（有延迟）分别跑1000个线程，一个线程循环10次，每次休眠100ms！
	         -- 测试次数三次
	         -- uuid从1开始。预期结果：uuid= 20000，结果20000正确！
	         -- 正确率 100%
        （2）两个IDE同时启动（有延迟）分别跑10000个线程，一个线程循环10次，每次休眠100ms！
            -- 测试次数三次
            -- uuid从1开始。预期结果：uuid= 200000，结果200000正确！



	2：使用三个IDE对同一个key进行测试，结论：当master宕机，没有完成的请求在master宕机时候都会失败，等待哨兵重新选定新的master后，
	   当再次发起请求，请求响应数据正常！

	     测试过程：
	     （1）第一步：两个IDE同时启动（有延迟）分别跑10个线程，一个线程循环100次，每次休眠500ms！
	         在运行期间，将redis的master服务杀掉（）
	         第二步：哨兵模式选举master成功后，可以在控制台看到输出，此时用第三个IDE，用同样的key去发送请求，若之前的mater宕机时候，uuid最后
	                输出值为 255！新的请求调用，输出的uuid值应该从6667开始！
	          ---测试三次
	          ---若uuid宕机时候为255，预期结果：256开始！结果：不是从256，可能是257开始，与前面的不重复！


	     注：有一种可能情况，当redismaster宕机时候，数据没有同步，slave升级为master时候，没有保存之前的数据，就会出现重复的uuid！




	3：性能：只是简单的demo示例进行测试，没有结合Spring项目和数据库进行测试。待完善补充！