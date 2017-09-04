package com.ame.model.interceptor;

import java.lang.reflect.Method;

import javax.servlet.http.HttpSession;

import org.apache.struts2.ServletActionContext;

import com.ame.systemutil.annotation.DownloadAnnotation;
import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.ActionProxy;
import com.opensymphony.xwork2.interceptor.MethodFilterInterceptor;

public class MethodInterceptor extends MethodFilterInterceptor{

	/*
	 * @date 2017-3-23
	 * @author YBF
	 * 拦截重复下载
	  STRUTS2拦截器要继承MethodFilterInterceptor，重写里面的doIntercept方法
	 */
	private static final long serialVersionUID = 1L;

	private static final String PRE_FIX = "download_";
	
	@Override
	protected String doIntercept(ActionInvocation invocation) throws Exception {
		Object action = invocation.getAction();
		ActionProxy actionProxy = invocation.getProxy();
		Method method = action.getClass().getMethod(actionProxy.getMethod());//获取拦截方法
		String methodValue = getMethodValue(method);//获取方法名
		if(methodValue == null){
			return invocation.invoke();
		}
		if(!getDownloadLock(ServletActionContext.getRequest().getSession(),methodValue)){//当session中有值时，说明真在下载
			//没能占有下载锁  要加提示
			System.out.println("没能占有下载锁  要加提示");
			ServletActionContext.getResponse().setHeader("Content-Type","text/html; charset=UTF-8");
			ServletActionContext.getResponse().getWriter().print("{message:'正在下载中,请勿重复操作'}");
			return null;
		}
		try{
			return invocation.invoke();
		}finally{
			//最终都要讲session中的值去除
			releaseDownloadLock(ServletActionContext.getRequest().getSession(),methodValue);
		}

	}

	/**
	 * @Description: 释放锁
	 * @param @param session
	 * @param @param methodValue   
	 * @return void  
	 * @throws
	 * @author KXL
	 * @date 2017-8-15
	 */
	private void releaseDownloadLock(HttpSession session,String methodValue) {
		synchronized (session) {
			String key = PRE_FIX + methodValue;
			session.removeAttribute(key);
		}
	}

	/**
	 * @Description: 获取拦截方法
	 * @param @param method
	 * @param @return   
	 * @return String  
	 * @throws
	 * @author KXL
	 * @date 2017-8-15
	   DownloadAnnotation为注解类，然后通过注解方式将注解类注解到具体你想拦截的方法上面
			例如：

			这是在assemblyBomInstanceAction中的一个方法，然后拦截器就会拦截该方法，并且获取到
			“assemblyBomInstanceAction_exportEwoBomTem”这个名称
			@DownloadAnnotation("assemblyBomInstanceAction_exportEwoBomTem")
			public String exportEwoBomTem(){
				try {
					exportProjectBomFileName="ewo_bom_temp.csv";
					String templateExcelPath = getServletContext().getRealPath("");
					exportProjectBomStream = MyIOUtils.getArrayStream(templateExcelPath + "/excelTemplate/ewo_bom_temp.csv");
				} catch (Exception e) {
					LOG.error(e.getMessage());
					error(handlerException(e,"导出失败!"));
				}
		return "exportEwoBomTem";
	}

	 */
	private String getMethodValue(Method method) {
		DownloadAnnotation download = method.getAnnotation(DownloadAnnotation.class);
		if(download != null){
			return download.value();
		}
		return null;
	}

	/**
	 * @param methodValue 拦截的方法名
	 * @param session 
	 * @Description: 占有锁
	 * @param    
	 * @return void  
	 * @throws
	 * @author KXL
	 * @date 2017-8-15
	 */
	private boolean getDownloadLock(HttpSession session, String methodValue) {
		synchronized (session) {
			String key = PRE_FIX + methodValue;
			Boolean lock = (Boolean) session.getAttribute(key);
			if(lock == null || !lock){
				session.setAttribute(key, true);
				return true;
			}
			return false;
		}
	}
}
