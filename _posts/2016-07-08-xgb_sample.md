---
layout:            post
title:             "Xgboost Sample Code"
date:              2016-07-08 20:00:00 +0800
tags:              DM
category:          Data Mining
---
*Coded by Kaggler [Misfyre](https://www.kaggle.com/misfyre)*

	# -*-coding: utf-8 -*-
	import numpy as np
	import xgboost as xgb
	import pandas as pd
	import math
	
	from sklearn.cross_validation import train_test_split
	from ml_metrics import rmsle
	
	print ('')
	print ('Loading Data...')
	
	def evalerror(preds, dtrain):
	
	    labels = dtrain.get_label()
	    assert len(preds) == len(labels)
	    labels = labels.tolist()
	    preds = preds.tolist()
	    terms_to_sum = [(math.log(labels[i] + 1) - math.log(max(0,preds[i]) + 1)) ** 2.0 for i,pred in enumerate(labels)]
	    return 'error', (sum(terms_to_sum) * (1.0/len(preds))) ** 0.5
	
	train = pd.read_csv('../input/train.csv', nrows = 7000000)  # nrows: number of rows to read(default: None)
	test = pd.read_csv('../input/test.csv')
	
	print ('')
	print ('Training_Shape:', train.shape)  # shape: return (rows, columns)
	
	ids = test['id']
	test = test.drop(['id'],axis = 1)  # axis=1 MEANS drop columns; axis=0 MEANS drop rows;
	
	y = train['Demanda_uni_equil']
	X = train[test.columns.values]  # Select attributes appears in test
	
	# [en] Devide the train set into one train set and one test set
	# [zh] 將訓練集隨機分出部分測試集，測試集占比為20%
	X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=1729)
	
	print ('Division_Set_Shapes:', X.shape, y.shape)
	print ('Validation_Set_Shapes:', X_train.shape, X_test.shape)
	
	params = {}
	params['objective'] = "reg:linear"
	params['eta'] = 0.025
	params['max_depth'] = 5
	params['subsample'] = 0.8
	params['colsample_bytree'] = 0.6
	params['silent'] = True
	
	print ('')
	
	test_preds = np.zeros(test.shape[0])
	xg_train = xgb.DMatrix(X_train, label=y_train)
	xg_test = xgb.DMatrix(X_test)
	
	watchlist = [(xg_train, 'train')]
	num_rounds = 100  # [zh] 迴圈次數
	
	# [zh] feval: 評判函數
	xgclassifier = xgb.train(params, xg_train, num_rounds, watchlist, feval = evalerror, early_stopping_rounds= 20, verbose_eval = 10)
	preds = xgclassifier.predict(xg_test, ntree_limit=xgclassifier.best_iteration)
	
	print ('RMSLE Score:', rmsle(y_test, preds))
	
	fxg_test = xgb.DMatrix(test)
	fold_preds = np.around(xgclassifier.predict(fxg_test, ntree_limit=xgclassifier.best_iteration), decimals = 1)  # around: 四捨五入
	test_preds += fold_preds
	
	submission = pd.DataFrame({'id':ids, 'Demanda_uni_equil': test_preds})
	submission.to_csv('submission.csv', index=False)