import { TestBed } from '@angular/core/testing';
import { CollateralTaskStepsMapperService } from './collateral-task-steps-mapper.service';
import { of, throwError } from 'rxjs';

describe('CollateralTaskStepsMapperService', () => {
  let service: CollateralTaskStepsMapperService;
  let mockAppStateMgmt: any;
  let mockTaskHttpService: any;
  let mockAppOverlayService: any;
  let mockRouter: any;

  beforeEach(() => {
    mockAppStateMgmt = {
      getCurrentTask: jasmine.createSpy().and.returnValue({ subTasks: ['task1', 'task2'] }),
      updateSubTasksStatusForDisposition: jasmine.createSpy(),
    };

    mockTaskHttpService = {
      dispositionProfileByTaskName: jasmine.createSpy().and.returnValue(of('SuccessResponse')),
    };

    mockAppOverlayService = {
      showSpinner: jasmine.createSpy(),
      hideSpinner: jasmine.createSpy(),
    };

    mockRouter = {
      navigate: jasmine.createSpy(),
    };

    TestBed.configureTestingModule({
      providers: [
        { provide: 'AppStateMgmtService', useValue: mockAppStateMgmt },
        { provide: 'TaskHttpService', useValue: mockTaskHttpService },
        { provide: 'AppOverlayService', useValue: mockAppOverlayService },
        { provide: 'Router', useValue: mockRouter },
      ],
    });

    service = TestBed.inject(CollateralTaskStepsMapperService);
    service['appStateMgmt'] = mockAppStateMgmt;
    service['taskHttpService'] = mockTaskHttpService;
    service['appOverlayService'] = mockAppOverlayService;
    service['router'] = mockRouter;
  });

  describe('triggerDispositionEvent', () => {
    it('should create a request object with taskId and subTasks when subTasks exist', () => {
      service.taskProfileConfig = { profileInfo: { taskId: '123' } };

      service.triggerDispositionEvent();

      expect(mockAppStateMgmt.updateSubTasksStatusForDisposition).toHaveBeenCalled();
      expect(mockTaskHttpService.dispositionProfileByTaskName).toHaveBeenCalledWith({
        taskId: '123',
        subTasks: ['task1', 'task2'],
      });
      expect(mockAppOverlayService.showSpinner).toHaveBeenCalled();
    });

    it('should create a request object without subTasks when no subTasks exist', () => {
      mockAppStateMgmt.getCurrentTask.and.returnValue(null);
      service.taskProfileConfig = { profileInfo: { taskId: '123' } };

      service.triggerDispositionEvent();

      expect(mockTaskHttpService.dispositionProfileByTaskName).toHaveBeenCalledWith({
        taskId: '123',
      });
    });

    it('should navigate to home on successful response', () => {
      service.taskProfileConfig = { profileInfo: { taskId: '123' } };

      service.triggerDispositionEvent();

      expect(mockRouter.navigate).toHaveBeenCalledWith(['/mfe-collateral-task-web', 'home']);
      expect(mockAppOverlayService.hideSpinner).toHaveBeenCalled();
    });

    it('should handle errors and navigate to error page', () => {
      service.taskProfileConfig = { profileInfo: { taskId: '123' } };
      mockTaskHttpService.dispositionProfileByTaskName.and.returnValue(throwError('Error'));

      spyOn(console, 'error');

      service.triggerDispositionEvent();

      expect(console.error).toHaveBeenCalledWith(
        'exception in getting Profile by taskName = ',
        'Error'
      );
      expect(mockRouter.navigate).toHaveBeenCalledWith(['500']);
      expect(mockAppOverlayService.hideSpinner).toHaveBeenCalled();
    });
  });
});


import { TestBed } from '@angular/core/testing';
import { CollateralTaskStepsMapperService } from './collateral-task-steps-mapper.service';

describe('CollateralTaskStepsMapperService', () => {
  let service: CollateralTaskStepsMapperService;
  let mockTaskValidationService: any;

  beforeEach(() => {
    mockTaskValidationService = {
      dataSetters: jasmine.createSpy(),
    };

    TestBed.configureTestingModule({
      providers: [
        { provide: 'TaskValidationService', useValue: mockTaskValidationService },
      ],
    });

    service = TestBed.inject(CollateralTaskStepsMapperService);
    service['taskValidationService'] = mockTaskValidationService;

    service.guidedTaskModel = {
      steps: [{ stepId: 1 }, { stepId: 2 }, { stepId: 3 }],
    };

    service.taskProfileConfig = {
      taskList: [{ stepId: 1 }, { stepId: 2 }, { stepId: 3 }],
    };

    service.taskModel = { guidedTaskModel: {} };
  });

  describe('removeCurrentStep', () => {
    it('should remove the specified step from guidedTaskModel and taskProfileConfig', () => {
      const stepConfig = { stepId: 2 };

      service.removeCurrentStep(stepConfig, null);

      expect(service.guidedTaskModel.steps).toEqual([{ stepId: 1 }, { stepId: 3 }]);
      expect(service.taskProfileConfig.taskList).toEqual([{ stepId: 1 }, { stepId: 3 }]);
    });

    it('should update stepId values after removal', () => {
      const stepConfig = { stepId: 1 };

      service.removeCurrentStep(stepConfig, null);

      expect(service.guidedTaskModel.steps).toEqual([{ stepId: 2 }, { stepId: 3 }]);
      expect(service.guidedTaskModel.steps[0].stepId).toBe(1);
      expect(service.guidedTaskModel.steps[1].stepId).toBe(2);
    });

    it('should call taskValidationService.dataSetters', () => {
      const stepConfig = { stepId: 2 };

      service.removeCurrentStep(stepConfig, null);

      expect(mockTaskValidationService.dataSetters).toHaveBeenCalledWith(
        service.guidedTaskModel,
        service.taskProfileConfig,
        service.taskInfoGetResponse
      );
    });
  });
});

import { TestBed } from '@angular/core/testing';
import { CollateralTaskStepsMapperService } from './collateral-task-steps-mapper.service';

describe('CollateralTaskStepsMapperService', () => {
  let service: CollateralTaskStepsMapperService;
  let mockUtils: any;

  beforeEach(() => {
    mockUtils = {
      isEmpty: jasmine.createSpy(),
    };

    TestBed.configureTestingModule({
      providers: [
        { provide: 'UtilsService', useValue: mockUtils },
      ],
    });

    service = TestBed.inject(CollateralTaskStepsMapperService);
    service['utils'] = mockUtils;
    service.taskProfileConfig = {};
    spyOn(service, 'mapDocData');
  });

  describe('updateDocumentContainer', () => {
    it('should call mapDocData when docContainer is not empty', () => {
      service.taskProfileConfig.docContainer = { key: 'value' };
      mockUtils.isEmpty.and.returnValue(false);

      service.updateDocumentContainer();

      expect(mockUtils.isEmpty).toHaveBeenCalledWith(service.taskProfileConfig.docContainer);
      expect(service.mapDocData).toHaveBeenCalledWith(service.taskProfileConfig.docContainer);
    });

    it('should not call mapDocData when docContainer is empty', () => {
      service.taskProfileConfig.docContainer = {};
      mockUtils.isEmpty.and.returnValue(true);

      service.updateDocumentContainer();

      expect(mockUtils.isEmpty).toHaveBeenCalledWith(service.taskProfileConfig.docContainer);
      expect(service.mapDocData).not.toHaveBeenCalled();
    });

    it('should not throw an error when taskProfileConfig is undefined', () => {
      service.taskProfileConfig = undefined;
      mockUtils.isEmpty.and.returnValue(true);

      expect(() => service.updateDocumentContainer()).not.toThrow();
      expect(mockUtils.isEmpty).toHaveBeenCalledWith(undefined);
      expect(service.mapDocData).not.toHaveBeenCalled();
    });
  });
});

import { TestBed } from '@angular/core/testing';
import { CollateralTaskStepsMapperService } from './collateral-task-steps-mapper.service';

describe('CollateralTaskStepsMapperService', () => {
  let service: CollateralTaskStepsMapperService;
  let mockUtils: any;
  let mockTaskHydrateDataService: any;

  beforeEach(() => {
    mockUtils = {
      isEmpty: jasmine.createSpy(),
    };

    mockTaskHydrateDataService = {
      getObjectValue: jasmine.createSpy(),
      hydrateData: jasmine.createSpy(),
    };

    TestBed.configureTestingModule({
      providers: [
        { provide: 'UtilsService', useValue: mockUtils },
        { provide: 'TaskHydrateDataService', useValue: mockTaskHydrateDataService },
      ],
    });

    service = TestBed.inject(CollateralTaskStepsMapperService);
    service['utils'] = mockUtils;
    service['taskHydrateDataService'] = mockTaskHydrateDataService;
  });

  describe('mapDocData', () => {
    it('should not process if docContainer or dataMapModel is empty', () => {
      service.mapDocData(null);
      service.mapDocData({});

      expect(mockUtils.isEmpty).toHaveBeenCalled();
      expect(mockTaskHydrateDataService.getObjectValue).not.toHaveBeenCalled();
      expect(mockTaskHydrateDataService.hydrateData).not.toHaveBeenCalled();
    });

    it('should process DOCLIST type correctly', () => {
      const mockDocContainer = {
        dataMapModel: [{ type: 'DOCLIST', name: 'doc1' }],
      };
      const mockDataMap = mockDocContainer.dataMapModel[0];

      mockUtils.isEmpty.and.returnValue(false);
      mockTaskHydrateDataService.getObjectValue.and.returnValue('mockValue');

      service.mapDocData(mockDocContainer);

      expect(mockTaskHydrateDataService.getObjectValue).toHaveBeenCalledWith(mockDocContainer, mockDataMap.name);
      expect(mockTaskHydrateDataService.hydrateData).toHaveBeenCalledWith(mockDataMap, service.taskInfoGetResponse);
    });

    it('should process DOCDESC type correctly', () => {
      const mockDocContainer = {
        dataMapModel: [{ type: 'DOCDESC', name: 'path.to.value' }],
      };
      const mockDataMap = mockDocContainer.dataMapModel[0];

      mockUtils.isEmpty.and.returnValue(false);

      service.mapDocData(mockDocContainer);

      expect(mockTaskHydrateDataService.getObjectValue).toHaveBeenCalled();
    });

    it('should process NSHDATA type correctly', () => {
      const mockDocContainer = {
        dataMapModel: [{ type: 'NSHDATA', name: 'nshData' }],
        notesAndStipulationHistory: {},
      };
      const mockDataMap = mockDocContainer.dataMapModel[0];

      mockUtils.isEmpty.and.returnValue(false);
      mockTaskHydrateDataService.getObjectValue.and.returnValue(null);

      service.mapDocData(mockDocContainer);

      expect(mockDocContainer.notesAndStipulationHistory['notesAndStipHistoryData']).toEqual([]);
    });

    it('should log message when document list is not available', () => {
      spyOn(console, 'log');
      const mockDocContainer = {
        dataMapModel: [{ type: 'DOCLIST', name: 'doc1' }],
      };

      mockUtils.isEmpty.and.returnValue(true);

      service.mapDocData(mockDocContainer);

      expect(console.log).toHaveBeenCalledWith('DocumentList info is not available in subtask config');
    });
  });
});

