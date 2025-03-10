
import { CollateralTaskStepsMapperService } from '../path-to-service';  // Adjust import path

describe('CollateralTaskStepsMapperService - mapDocData', () => {
  let service: CollateralTaskStepsMapperService;

  beforeEach(() => {
    service = new CollateralTaskStepsMapperService();
  });

  it('should process CDATA correctly when all values are provided', () => {
    const docContainer = {};
    const dataMap = {
      type: "CDATA",
      key: "sampleKey",
      identifiers: ["id1", "id2"],
      value: {
        id1: "value1",
        id2: "value2"
      },
      headers: [
        { key: "header1", valueType: "NAME", value: "headerValue1" },
        { key: "header2", valueType: "NAMES", value: "headerValue2" }
      ]
    };

    service.mapDocData(docContainer, [dataMap]);

    // Verify data was processed correctly
    expect(docContainer["sampleKey"]).toBeDefined();
    expect(docContainer["sampleKey"].header1).toEqual(["ProcessedName"]);
    expect(docContainer["sampleKey"].header2).toEqual(["ProcessedNames"]);
  });

  it('should skip processing when dataMap.key is missing', () => {
    const docContainer = {};
    const dataMap = {
      type: "CDATA",
      identifiers: ["id1", "id2"],  // No key provided
      value: { id1: "value1", id2: "value2" }
    };

    service.mapDocData(docContainer, [dataMap]);

    expect(docContainer).toEqual({});  // Should not modify docContainer
  });

  it('should skip processing when dataMap.identifiers is empty', () => {
    const docContainer = {};
    const dataMap = {
      type: "CDATA",
      key: "sampleKey",
      identifiers: [],  // Empty identifiers
      value: { id1: "value1", id2: "value2" }
    };

    service.mapDocData(docContainer, [dataMap]);

    expect(docContainer).toEqual({});
  });

  it('should correctly handle missing or empty values in dataMap.value', () => {
    const docContainer = {};
    const dataMap = {
      type: "CDATA",
      key: "sampleKey",
      identifiers: ["id1", "id2"],
      value: {}  // Empty value object
    };

    service.mapDocData(docContainer, [dataMap]);

    expect(docContainer["sampleKey"]).toEqual({});
  });

  it('should correctly process header information', () => {
    const docContainer = {};
    const dataMap = {
      type: "CDATA",
      key: "sampleKey",
      identifiers: ["id1"],
      value: { id1: "testValue" },
      headers: [
        { key: "headerA", valueType: "NAME", value: "headerValueA" },
        { key: "headerB", valueType: "NAMES", value: "headerValueB" },
        { key: "headerC", valueType: "DEFAULT", value: "headerValueC" }
      ]
    };

    service.mapDocData(docContainer, [dataMap]);

    expect(docContainer["sampleKey"].headerA).toEqual(["ProcessedName"]);
    expect(docContainer["sampleKey"].headerB).toEqual(["ProcessedNames"]);
    expect(docContainer["sampleKey"].headerC).toEqual(["headerValueC"]);
  });

  it('should correctly handle missing headers', () => {
    const docContainer = {};
    const dataMap = {
      type: "CDATA",
      key: "sampleKey",
      identifiers: ["id1"],
      value: { id1: "testValue" },
      headers: []  // No headers provided
    };

    service.mapDocData(docContainer, [dataMap]);

    expect(docContainer["sampleKey"]).toEqual({});
  });

  it('should correctly process multiple dataMap entries', () => {
    const docContainer = {};
    const dataMaps = [
      {
        type: "CDATA",
        key: "key1",
        identifiers: ["id1"],
        value: { id1: "value1" },
        headers: [{ key: "header1", valueType: "NAME", value: "headerValue1" }]
      },
      {
        type: "CDATA",
        key: "key2",
        identifiers: ["id2"],
        value: { id2: "value2" },
        headers: [{ key: "header2", valueType: "NAMES", value: "headerValue2" }]
      }
    ];

    service.mapDocData(docContainer, dataMaps);

    expect(docContainer["key1"].header1).toEqual(["ProcessedName"]);
    expect(docContainer["key2"].header2).toEqual(["ProcessedNames"]);
  });

});
