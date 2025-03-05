describe('removeCurrentStep', () => {
  beforeEach(() => {
    // Mock objects
    service.guidedTaskModel = { steps: [{ stepId: 1 }, { stepId: 2 }] };
    service.taskProfileConfig = { taskList: [{ stepId: 1 }, { stepId: 2 }] };
    service.taskModel = { guidedTaskModel: null };
    service.taskValidationService = { dataSetters: jasmine.createSpy('dataSetters') };
  });

  it('should remove the current step from guidedTaskModel and taskProfileConfig', () => {
    const stepConfig = { stepId: 1 };
    const keyImpactedByIteration = 'testKey';

    service.removeCurrentStep(stepConfig, keyImpactedByIteration);

    expect(service.guidedTaskModel.steps.length).toBe(1);
    expect(service.taskProfileConfig.taskList.length).toBe(1);
    expect(service.taskValidationService.dataSetters).toHaveBeenCalledWith(
      service.guidedTaskModel,
      service.taskProfileConfig,
      service.taskInfoGetResponse
    );
    expect(service.taskModel.guidedTaskModel).toBe(service.guidedTaskModel);
  });

  it('should update stepId indexes after removal', () => {
    const stepConfig = { stepId: 1 };
    const keyImpactedByIteration = 'testKey';

    service.removeCurrentStep(stepConfig, keyImpactedByIteration);

    expect(service.guidedTaskModel.steps[0].stepId).toBe(1);
    expect(service.taskProfileConfig.taskList[0].stepId).toBe(1);
  });

  it('should do nothing if stepConfig is missing stepId', () => {
    const stepConfig = {};
    const keyImpactedByIteration = 'testKey';

    service.removeCurrentStep(stepConfig, keyImpactedByIteration);

    expect(service.guidedTaskModel.steps.length).toBe(2);
    expect(service.taskProfileConfig.taskList.length).toBe(2);
  });

  it('should handle an empty guidedTaskModel.steps array gracefully', () => {
    service.guidedTaskModel.steps = [];
    const stepConfig = { stepId: 1 };
    const keyImpactedByIteration = 'testKey';

    service.removeCurrentStep(stepConfig, keyImpactedByIteration);

    expect(service.guidedTaskModel.steps.length).toBe(0);
  });

  it('should handle an empty taskProfileConfig.taskList array gracefully', () => {
    service.taskProfileConfig.taskList = [];
    const stepConfig = { stepId: 1 };
    const keyImpactedByIteration = 'testKey';

    service.removeCurrentStep(stepConfig, keyImpactedByIteration);

    expect(service.taskProfileConfig.taskList.length).toBe(0);
  });
});
---------------------------------------------------------------------------------------
describe('combineStepComponentData', () => {
  it('should combine namesArray and partyIDs into key-value pairs when both arrays are valid and of equal length', () => {
    const namesArray = ['John', 'Doe', 'Alice'];
    const partyIDs = ['123', '456', '789'];

    const result = service.combineStepComponentData(namesArray, partyIDs);

    expect(result).toEqual([
      { key: '123', value: 'John' },
      { key: '456', value: 'Doe' },
      { key: '789', value: 'Alice' },
    ]);
  });

  it('should return an empty array when namesArray and partyIDs are not arrays', () => {
    const namesArray = 'invalid';
    const partyIDs = 123;

    spyOn(console, 'error');

    const result = service.combineStepComponentData(namesArray as any, partyIDs as any);

    expect(result).toEqual([]);
    expect(console.error).toHaveBeenCalledWith(
      'The arrays are not of the same length or not arrays.'
    );
  });

  it('should return an empty array when namesArray and partyIDs have different lengths', () => {
    const namesArray = ['John', 'Doe'];
    const partyIDs = ['123']; // Mismatched length

    spyOn(console, 'error');

    const result = service.combineStepComponentData(namesArray, partyIDs);

    expect(result).toEqual([]);
    expect(console.error).toHaveBeenCalledWith(
      'The arrays are not of the same length or not arrays.'
    );
  });

  it('should return an empty array when either namesArray or partyIDs is null or undefined', () => {
    spyOn(console, 'error');

    expect(service.combineStepComponentData(null as any, ['123'])).toEqual([]);
    expect(service.combineStepComponentData(['John'], undefined as any)).toEqual([]);
    expect(service.combineStepComponentData(undefined as any, undefined as any)).toEqual([]);

    expect(console.error).toHaveBeenCalledTimes(3);
  });

  it('should return an empty array when both arrays are empty', () => {
    const namesArray: string[] = [];
    const partyIDs: string[] = [];

    const result = service.combineStepComponentData(namesArray, partyIDs);

    expect(result).toEqual([]); // Should return an empty array
  });
});

---------------------------------------------------------------------------------------
describe('triggerDispositionEvent', () => {
  let service: CollateralTaskStepsMapperService;
  let appStateMgmtMock: any;
  let taskHttpServiceMock: any;
  let appOverlayServiceMock: any;
  let routerMock: any;

  beforeEach(() => {
    appStateMgmtMock = {
      getCurrentTask: jasmine.createSpy('getCurrentTask').and.returnValue({ subTasks: ['subTask1'] }),
      updateSubTasksStatusForDisposition: jasmine.createSpy('updateSubTasksStatusForDisposition'),
    };

    taskHttpServiceMock = {
      dispositionProfileByTaskName: jasmine.createSpy('dispositionProfileByTaskName').and.returnValue({
        subscribe: (success: Function, error: Function) => {
          service.handleDispositionSuccess = success;
          service.handleDispositionError = error;
        },
      }),
    };

    appOverlayServiceMock = {
      showSpinner: jasmine.createSpy('showSpinner'),
      hideSpinner: jasmine.createSpy('hideSpinner'),
    };

    routerMock = {
      navigate: jasmine.createSpy('navigate'),
    };

    service = new CollateralTaskStepsMapperService(
      appStateMgmtMock,
      taskHttpServiceMock,
      appOverlayServiceMock,
      routerMock
    );
  });

  it('should send a disposition request and update subTasks when getCurrentTask has subTasks', () => {
    service.triggerDispositionEvent();

    expect(appStateMgmtMock.getCurrentTask).toHaveBeenCalled();
    expect(appStateMgmtMock.updateSubTasksStatusForDisposition).toHaveBeenCalled();
    expect(taskHttpServiceMock.dispositionProfileByTaskName).toHaveBeenCalledWith({
      taskId: service.taskProfileConfig.profileInfo.taskId,
      subTasks: ['subTask1'],
    });
    expect(appOverlayServiceMock.showSpinner).toHaveBeenCalled();
  });

  it('should send a disposition request with no subTasks when getCurrentTask is null', () => {
    appStateMgmtMock.getCurrentTask.and.returnValue(null);

    service.triggerDispositionEvent();

    expect(taskHttpServiceMock.dispositionProfileByTaskName).toHaveBeenCalledWith({
      taskId: service.taskProfileConfig.profileInfo.taskId,
    });
  });

  it('should navigate to the correct route on successful disposition response', () => {
    service.triggerDispositionEvent();
    service.handleDispositionSuccess({ success: true });

    expect(appOverlayServiceMock.hideSpinner).toHaveBeenCalled();
    expect(routerMock.navigate).toHaveBeenCalledWith(['/mfe-collateral-task-web', 'home']);
  });

  it('should handle errors correctly and navigate to 500 on failure', () => {
    spyOn(console, 'error');

    service.triggerDispositionEvent();
    service.handleDispositionError(new Error('Network error'));

    expect(console.error).toHaveBeenCalledWith('exception in getting Profile by taskName : ', jasmine.any(Error));
    expect(routerMock.navigate).toHaveBeenCalledWith(['500']);
    expect(appOverlayServiceMock.hideSpinner).toHaveBeenCalled();
  });

  it('should still work if getCurrentTask() is undefined', () => {
    appStateMgmtMock.getCurrentTask.and.returnValue(undefined);

    service.triggerDispositionEvent();

    expect(appStateMgmtMock.getCurrentTask).toHaveBeenCalled();
    expect(appStateMgmtMock.updateSubTasksStatusForDisposition).not.toHaveBeenCalled();
    expect(taskHttpServiceMock.dispositionProfileByTaskName).toHaveBeenCalledWith({
      taskId: service.taskProfileConfig.profileInfo.taskId,
    });
  });
});

