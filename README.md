
it('should trigger browser events and navigate to home on valid request', (done) => {
  const mockContextService = {
    getUserContext: () => ({ userName: 'mockUser' })
  };

  const mockLoggerService = {
    log: jasmine.createSpy('log')
  };

  const mockAppStateMgmt = {
    getCurrentTask: () => ({ subTasks: ['sub1', 'sub2'] }),
    updateSubTaskForBrowserEvents: jasmine.createSpy('updateSubTaskForBrowserEvents')
  };

  const mockTaskProfileConfig = {
    profileInfo: {
      taskId: '123',
      taskName: 'Task Name',
      userId: 'user123'
    }
  };

  const mockTaskStepService = {
    dispositionData: jasmine.createSpy('dispositionData').and.returnValue({
      subscribe: (callbacks: any) => {
        callbacks.next({}); // simulate observable next
      }
    })
  };

  const mockRouter = {
    navigate: jasmine.createSpy('navigate')
  };

  service = new CollateralTaskStepsMapperService(
    {} as any,
    {} as any,
    {} as any,
    {} as any,
    mockLoggerService as any,
    {} as any,
    mockContextService as any,
    {} as any,
    mockTaskStepService as any,
    {} as any,
    {} as any
  );

  (service as any).taskProfileConfig = mockTaskProfileConfig;
  (service as any).appStateMgmt = mockAppStateMgmt;
  (service as any).router = mockRouter;

  service.triggerBrowserEventsForDisposition().then(() => {
    expect(mockTaskStepService.dispositionData).toHaveBeenCalled();
    expect(mockRouter.navigate).toHaveBeenCalledWith(['/mfe-collateral-task-web', 'home']);
    expect(mockLoggerService.log).toHaveBeenCalled();
    done();
  });
});
