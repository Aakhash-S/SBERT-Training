describe('mapDocData - NSHDATA Case', () => {
  let service: CollateralTaskStepsMapperService;
  let docContainer: any;

  beforeEach(() => {
    service = new CollateralTaskStepsMapperService();
    docContainer = { notesAndStipulationHistory: {} };
  });

  it('should populate notesAndStipHistoryData when getModelObj and getObjValue are not empty', () => {
    const getModelObj = [{ field: 'value' }];
    const getObjValue = [{ rowData: 'testData' }];
    spyOn(service.utils, 'isEmpty').and.callFake((input) => !input || input.length === 0);
    spyOn(service.taskHydrateDataService, 'getObjectValue').and.returnValue(getModelObj);

    service.mapDocData({
      dataMapModel: [{ type: 'NSHDATA', name: 'someData' }],
      docContainer,
    });

    expect(docContainer.notesAndStipulationHistory.notesAndStipHistoryData).toBeDefined();
    expect(docContainer.notesAndStipulationHistory.notesAndStipHistoryData.length).toBeGreaterThan(0);
  });

  it('should set notesAndStipHistoryData to an empty array if getObjValue is empty', () => {
    spyOn(service.utils, 'isEmpty').and.callFake((input) => !input || input.length === 0);
    spyOn(service.taskHydrateDataService, 'getObjectValue').and.returnValue([]);

    service.mapDocData({
      dataMapModel: [{ type: 'NSHDATA', name: 'someData' }],
      docContainer,
    });

    expect(docContainer.notesAndStipulationHistory.notesAndStipHistoryData).toEqual([]);
  });

  it('should set notesAndStipHistoryData to an empty array if getModelObj is empty', () => {
    spyOn(service.utils, 'isEmpty').and.callFake((input) => !input || input.length === 0);

    service.mapDocData({
      dataMapModel: [{ type: 'NSHDATA', name: 'someData' }],
      docContainer,
    });

    expect(docContainer.notesAndStipulationHistory.notesAndStipHistoryData).toEqual([]);
  });
});
