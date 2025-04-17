
describe('triggerBrowserEventsForDisposition', () => {
  it('should call dispositionData and navigate on success', (done) => {
    const request = {
      taskId: mockDocumentValidstionTaskProfile.profileInfo.taskId,
      taskName: mockDocumentValidstionTaskProfile.profileInfo.taskName,
      userId: mockDocumentValidstionTaskProfile.profileInfo.userId,
      subTasks: mockAppState.getCurrentTask().subTasks
    };

    spyOn(component['appStateMgmt'], 'updateSubTaskForBrowserEvents');
    spyOn(component['loggerService'], 'log');
    spyOn(component['taskStepService'], 'dispositionData').and.returnValue(of({}));
    spyOn(component['approvalLayerService'], 'hideSpinner');
    spyOn(component['router'], 'navigate');

    component.triggerBrowserEventsForDisposition().then(() => {
      expect(component['taskStepService'].dispositionData).toHaveBeenCalledWith(request);
      expect(component['router'].navigate).toHaveBeenCalledWith(['/nfe-collateral-task-web', 'home']);
      expect(component['approvalLayerService'].hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should navigate to 500 and hide spinner on error', (done) => {
    spyOn(component['taskStepService'], 'dispositionData').and.returnValue(throwError(() => new Error('Error')));
    spyOn(component['router'], 'navigate');
    spyOn(component['approvalLayerService'], 'hideSpinner');
    spyOn(console, 'error');

    component.triggerBrowserEventsForDisposition().catch(() => {
      expect(console.error).toHaveBeenCalled();
      expect(component['router'].navigate).toHaveBeenCalledWith(['/500']);
      expect(component['approvalLayerService'].hideSpinner).toHaveBeenCalled();
      done();
    });
  });

  it('should resolve immediately when profileInfo is not available', (done) => {
    component['taskProfileConfig'].profileInfo = null as any;
    spyOn(component, 'hideLeftNavBar');
    component.triggerBrowserEventsForDisposition().then(() => {
      expect(component.hideLeftNavBar).toHaveBeenCalledWith(false);
      done();
    });
  });
});
