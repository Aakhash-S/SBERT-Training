describe('triggerBrowserEventsForDisposition', () => {
  let service: any;

  const mockTaskStepService = {
    dispositionData: jasmine.createSpy()
  };

  const mockLoggerService = {
    log: jasmine.createSpy()
  };

  const mockApprovalLayerService = {
    hideSpinner: jasmine.createSpy()
  };

  const mockRouter = {
    navigate: jasmine.createSpy()
  };

  const mockAppStateMgmt = {
    getCurrentTask: jasmine.createSpy().and.returnValue({ subTasks: ['sub1'] }),
    updateSubtaskForBrowserEvents: jasmine.createSpy()
  };

  beforeEach(() => {
    service = new CollateralTaskStepsMapperService(
      {}, // other args if needed
      {}, 
      mockTaskStepService,
      {},
      mockLoggerService,
      mockRouter,
      mockAppStateMgmt,
      mockApprovalLayerService
    );

    service.taskProfileConfig = {
      profileInfo: {
        taskId: '123',
        taskName: 'Some Task',
        userId: 'user123'
      }
    };

    service.contextService = {
      getUserContext: () => ({ userName: 'testUser' })
    };
  });

  it('should call dispositionData and navigate on success', (done) => {
    const expectedRequest = {
      taskId: '123',
      taskName: 'Some Task',
      userId: 'user123',
      subTasks: ['sub1']
    };

    mockTaskStepService.dispositionData.and.returnValue(of({}));

    service.triggerBrowserEventsForDisposition().then(() => {
      expect(mockTaskStepService.dispositionData).toHaveBeenCalledWith(expectedRequest);
      expect(mockRouter.navigate).toHaveBeenCalledWith(['/nfe-collateral-task-web', 'home']);
      expect(mockApprovalLayerService.hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should navigate to 500 and hide spinner on error', (done) => {
    mockTaskStepService.dispositionData.and.returnValue(throwError(() => new Error('Error')));
    spyOn(console, 'error');

    service.triggerBrowserEventsForDisposition().catch(() => {
      expect(console.error).toHaveBeenCalled();
      expect(mockRouter.navigate).toHaveBeenCalledWith(['/500']);
      expect(mockApprovalLayerService.hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should resolve immediately when profileInfo is not available', (done) => {
    service.taskProfileConfig.profileInfo = null;
    spyOn(service, 'hideLeftNavBar');

    service.triggerBrowserEventsForDisposition().then(() => {
      expect(service.hideLeftNavBar).toHaveBeenCalledWith(false);
      done();
    });
  });
});
